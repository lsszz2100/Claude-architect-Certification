# Chapter 13: 구조화된 출력 설계

> 📅 2026년 04월 05일 기준  
> 🎯 **Domain 4 — tool_use가 가장 신뢰할 수 있는 방법**

---

## 13.1 tool_use와 JSON 스키마

> 🎯 **시험 핵심: tool_use = 구문 오류 제거, 의미 오류는 별도 검증**

### 왜 tool_use로 구조화 출력을?

```python
# ❌ 방법 1: 텍스트 응답에서 JSON 파싱 (신뢰할 수 없음)
response = "다음 JSON으로 추출했습니다:\n```json\n{...}\n```"
# → 마크다운, 앞뒤 텍스트, 잘못된 JSON 등으로 파싱 실패 가능

# ❌ 방법 2: "JSON으로만 응답해줘" 프롬프트
# → 때로는 설명 텍스트가 앞에 붙어 파싱 실패

# ✅ 방법 3: tool_use (가장 신뢰할 수 있음)
# → Claude가 반드시 스키마를 따르는 구조화된 객체로 응답
# → JSON 구문 오류 자동 제거
```

### tool_use로 구조화 출력 구현

```python
import anthropic
import json

client = anthropic.Anthropic()

# 추출 툴 정의
extraction_tool = {
    "name": "extract_invoice_data",
    "description": "송장에서 구조화된 데이터를 추출합니다",
    "input_schema": {
        "type": "object",
        "properties": {
            "invoice_number": {
                "type": "string",
                "description": "송장 번호"
            },
            "invoice_date": {
                "type": ["string", "null"],  # nullable!
                "description": "송장 발행일 (YYYY-MM-DD 형식)"
            },
            "due_date": {
                "type": ["string", "null"],
                "description": "지불 기한 (없을 수 있음)"
            },
            "total_amount": {
                "type": "number",
                "description": "총 금액"
            },
            "currency": {
                "type": "string",
                "enum": ["KRW", "USD", "EUR", "JPY", "other"],
                "description": "통화"
            },
            "currency_detail": {
                "type": ["string", "null"],
                "description": "currency가 'other'인 경우 통화명"
            },
            "is_approximate": {
                "type": "boolean",
                "description": "금액이 대략적인 경우 true"
            }
        },
        "required": ["invoice_number", "total_amount", "currency"]
    }
}

def extract_invoice(invoice_text: str) -> dict:
    """송장에서 데이터 추출"""
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        tools=[extraction_tool],
        tool_choice={"type": "tool", "name": "extract_invoice_data"},
        messages=[{
            "role": "user",
            "content": f"다음 송장에서 정보를 추출해주세요:\n\n{invoice_text}"
        }]
    )
    
    # tool_use 결과 추출
    for block in response.content:
        if block.type == "tool_use" and block.name == "extract_invoice_data":
            return block.input
    
    return None
```

---

## 13.2 스키마 설계 원칙

### 1. Nullable 필드 활용

```python
# ❌ 잘못된 설계: 항상 존재해야 하는 required 필드
"required": ["invoice_number", "due_date"]  # due_date가 없는 송장도 있음!
# → Claude가 due_date를 만들어낼 수 있음 (환각)

# ✅ 올바른 설계: 없을 수 있는 필드는 nullable
"properties": {
    "due_date": {
        "type": ["string", "null"],  # null 허용
        "description": "지불 기한. 명시되지 않은 경우 null"
    }
}
```

### 2. enum + "other" 패턴

```python
# 확장 가능한 카테고리 설계
"category": {
    "type": "string",
    "enum": ["bug", "security", "performance", "style", "other"],
    "description": "이슈 카테고리"
},
"category_detail": {
    "type": ["string", "null"],
    "description": "category가 'other'인 경우 상세 설명"
}
```

### 3. 형식 정규화 규칙 포함

```python
# 프롬프트에 형식 정규화 규칙 추가
system_prompt = """
데이터를 추출할 때 다음 형식으로 정규화하세요:
- 날짜: YYYY-MM-DD 형식 (예: "2024년 1월 15일" → "2024-01-15")
- 금액: 숫자만 (단위 제거, 예: "1,500원" → 1500)
- 전화번호: +국가코드-지역-번호 (예: "010-1234-5678" → "+82-10-1234-5678")
"""
```

---

## 13.3 검증과 재시도 루프

### 의미적 검증 (Semantic Validation)

> ⚠️ tool_use는 구문 오류를 제거하지만, 의미 오류는 별도 검증 필요

```python
def validate_invoice_data(data: dict) -> list[str]:
    """의미적 검증: 값들이 논리적으로 올바른가?"""
    errors = []
    
    # 항목 합계가 총액과 일치하는지
    if "line_items" in data:
        calculated_total = sum(item["amount"] for item in data["line_items"])
        if abs(calculated_total - data["total_amount"]) > 0.01:
            errors.append(
                f"항목 합계({calculated_total})가 총액({data['total_amount']})과 다릅니다"
            )
    
    # 발행일이 지불기한보다 이른지
    if data.get("invoice_date") and data.get("due_date"):
        if data["invoice_date"] > data["due_date"]:
            errors.append("발행일이 지불기한보다 늦습니다")
    
    return errors


def extract_with_retry(invoice_text: str, max_retries: int = 2) -> dict:
    """검증 실패 시 오류 피드백과 함께 재시도"""
    
    messages = [{
        "role": "user",
        "content": f"다음 송장에서 정보를 추출해주세요:\n\n{invoice_text}"
    }]
    
    for attempt in range(max_retries + 1):
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            tools=[extraction_tool],
            tool_choice={"type": "tool", "name": "extract_invoice_data"},
            messages=messages
        )
        
        # 결과 추출
        result = None
        for block in response.content:
            if block.type == "tool_use":
                result = block.input
                break
        
        if not result:
            continue
        
        # 검증
        errors = validate_invoice_data(result)
        if not errors:
            return result  # 성공!
        
        if attempt < max_retries:
            # 오류 피드백과 함께 재시도
            messages.append({"role": "assistant", "content": response.content})
            messages.append({
                "role": "user",
                "content": f"""
이전 추출 결과에 다음 오류가 있습니다:
{chr(10).join(f'- {e}' for e in errors)}

원본 송장을 다시 확인하고 수정해주세요:
{invoice_text}
"""
            })
    
    return result  # 최선의 결과 반환


def is_retry_useful(error: str, document: str) -> bool:
    """재시도가 도움이 될지 판단"""
    
    # ✅ 재시도로 해결 가능: 형식 불일치
    if "형식이 올바르지 않습니다" in error:
        return True
    
    # ❌ 재시도로 해결 불가: 정보 자체가 없음
    if "원본 문서에 정보가 없습니다" in error:
        return False  # 외부 문서를 제공하지 않는 한 재시도 무의미
    
    return True
```

---

## 📝 챕터 요약

- tool_use: JSON 구문 오류 제거 → 가장 신뢰할 수 있는 구조화 출력
- nullable 필드: 없을 수 있는 정보에 null 허용 (환각 방지)
- enum + "other" + detail: 확장 가능한 카테고리 설계
- tool_use는 구문 오류만 제거, 의미적 오류(합계 불일치 등)는 별도 검증 필요
- 재시도: 형식 오류는 유효, 정보 자체 부재는 무효

---

> 🔗 다음 챕터: [배치 처리와 리뷰 아키텍처](14_batch_review.md)
