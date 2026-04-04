# Chapter 22: 시나리오 6 — 데이터 추출 파이프라인

> 📅 2026년 04월 05일 기준  
> 🎯 실제 시험 시나리오 6 해설

---

## 시나리오 개요

> 당신은 비정형 문서(이메일, PDF, 스캔 문서)에서 구조화된 데이터를 추출하는 시스템을 구축합니다.
> 다양한 형식의 문서에서 일관된 데이터를 추출해야 합니다.
> 추출 정확도와 환각(hallucination) 방지가 핵심 과제입니다.

---

## 핵심 과제: 환각 방지

### nullable 필드 설계

```python
import anthropic
import json

client = anthropic.Anthropic()

# ❌ 잘못된 스키마: nullable 없음
BAD_SCHEMA = {
    "type": "object",
    "properties": {
        "invoice_number": {"type": "string"},
        "vendor_name": {"type": "string"},
        "total_amount": {"type": "number"},
        "tax_rate": {"type": "number"},    # ← 항상 있다고 가정
        "discount_code": {"type": "string"}  # ← 항상 있다고 가정
    },
    "required": ["invoice_number", "vendor_name", "total_amount", "tax_rate", "discount_code"]
}
# 문제: tax_rate나 discount_code가 없으면 LLM이 값을 만들어냄 (환각)


# ✅ 올바른 스키마: nullable 필드 사용
GOOD_SCHEMA = {
    "type": "object",
    "properties": {
        "invoice_number": {"type": "string"},        # 필수
        "vendor_name": {"type": "string"},            # 필수
        "total_amount": {"type": "number"},           # 필수
        "tax_rate": {
            "type": ["number", "null"],               # ← nullable
            "description": "세율 %. 명시되지 않은 경우 null"
        },
        "discount_code": {
            "type": ["string", "null"],              # ← nullable
            "description": "할인 코드. 없으면 null"
        },
        "payment_due_date": {
            "type": ["string", "null"],              # ← nullable
            "format": "date",
            "description": "지불 기한. 찾을 수 없으면 null"
        }
    },
    "required": ["invoice_number", "vendor_name", "total_amount",
                 "tax_rate", "discount_code", "payment_due_date"]
    # required에 포함되지만 null 허용
}
```

---

## 구조화된 출력 구현

### tool_use 기반 추출

```python
EXTRACTION_TOOLS = [
    {
        "name": "extract_invoice_data",
        "description": """인보이스 문서에서 구조화된 데이터를 추출합니다.
        
        반드시 이 툴을 사용하여 추출 결과를 반환하세요.
        정보를 찾을 수 없는 경우 null을 사용하세요. 절대 추측하지 마세요.
        
        주의사항:
        - 금액은 항상 숫자로 변환 (통화 기호 제거)
        - 날짜는 ISO 8601 형식 (YYYY-MM-DD)
        - 없는 정보는 반드시 null (빈 문자열 "" 사용 금지)""",
        "input_schema": GOOD_SCHEMA
    }
]


def extract_data_from_document(document_text: str) -> dict:
    """문서에서 데이터 추출 with tool_use"""
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        tools=EXTRACTION_TOOLS,
        tool_choice={"type": "tool", "name": "extract_invoice_data"},  # 툴 강제
        messages=[{
            "role": "user",
            "content": f"""다음 인보이스 문서에서 데이터를 추출하세요.
            
정보가 없거나 불명확한 경우 반드시 null을 반환하세요.
절대 추측하지 마세요.

문서:
{document_text}"""
        }]
    )
    
    # tool_use 응답에서 추출된 데이터 가져오기
    for block in response.content:
        if block.type == "tool_use":
            return block.input
    
    return {}
```

### 검증-재시도 루프

```python
def extract_with_validation(document_text: str, max_retries: int = 3) -> dict:
    """추출 + 유효성 검사 + 재시도 루프"""
    
    last_error = None
    
    for attempt in range(max_retries):
        # 추출 시도
        extracted = extract_data_from_document(document_text)
        
        # 유효성 검사
        validation_errors = validate_extraction(extracted)
        
        if not validation_errors:
            return extracted  # 성공
        
        # 재시도 시 이전 오류 피드백 포함
        last_error = validation_errors
        print(f"시도 {attempt + 1} 실패: {validation_errors}")
        
        # 오류 피드백으로 재시도
        document_text = f"""이전 추출에서 다음 오류가 발생했습니다:
{json.dumps(validation_errors, ensure_ascii=False)}

위 오류를 수정하여 다시 추출하세요:

{document_text}"""
    
    # 최대 재시도 후 부분 결과 반환
    return {
        "data": extracted,
        "validation_failed": True,
        "errors": last_error
    }


def validate_extraction(data: dict) -> list[str]:
    """추출된 데이터 유효성 검사"""
    
    errors = []
    
    # 필수 필드 확인
    if not data.get("invoice_number"):
        errors.append("invoice_number는 필수입니다 (null 불가)")
    
    if not data.get("vendor_name"):
        errors.append("vendor_name은 필수입니다 (null 불가)")
    
    # 금액 유효성
    total = data.get("total_amount")
    if total is not None:
        if not isinstance(total, (int, float)):
            errors.append(f"total_amount는 숫자여야 합니다 (현재: {type(total).__name__})")
        elif total < 0:
            errors.append("total_amount는 음수일 수 없습니다")
    
    # 날짜 형식
    due_date = data.get("payment_due_date")
    if due_date is not None:
        import re
        if not re.match(r'\d{4}-\d{2}-\d{2}', due_date):
            errors.append(f"payment_due_date는 YYYY-MM-DD 형식이어야 합니다 (현재: {due_date})")
    
    return errors
```

---

## 다양한 문서 형식 처리

### Few-Shot으로 다양한 형식 처리

```python
FEW_SHOT_EXAMPLES = [
    # 예시 1: 표준 인보이스
    {
        "document": """인보이스 #INV-2024-001
발행처: ABC Corp
합계: $1,500.00
세율: 10%
지불기한: 2024-03-15""",
        "extraction": {
            "invoice_number": "INV-2024-001",
            "vendor_name": "ABC Corp",
            "total_amount": 1500.00,
            "tax_rate": 10.0,
            "discount_code": None,
            "payment_due_date": "2024-03-15"
        }
    },
    # 예시 2: 할인 코드 없음
    {
        "document": """INVOICE
Number: 99-2024
From: XYZ Ltd
Total Amount: $850""",
        "extraction": {
            "invoice_number": "99-2024",
            "vendor_name": "XYZ Ltd",
            "total_amount": 850.00,
            "tax_rate": None,           # ← 명시 안 됨 → null
            "discount_code": None,       # ← 없음 → null
            "payment_due_date": None     # ← 없음 → null
        }
    }
]


def build_extraction_prompt(document: str) -> str:
    """Few-shot 예시가 포함된 추출 프롬프트"""
    
    examples_text = "\n\n".join([
        f"문서:\n{ex['document']}\n\n추출 결과:\n{json.dumps(ex['extraction'], ensure_ascii=False)}"
        for ex in FEW_SHOT_EXAMPLES
    ])
    
    return f"""다음 예시를 참고하여 인보이스 데이터를 추출하세요.
정보가 없으면 null을 사용하세요.

{examples_text}

---
새 문서:
{document}"""
```

---

## 빈 결과 vs 접근 실패 구분

```python
def process_document_result(raw_result: dict) -> dict:
    """빈 결과와 접근 실패를 명확히 구분"""
    
    # ❌ 모든 null을 같게 취급
    # if not result: return "데이터 없음"
    
    # ✅ 명확한 구분
    if raw_result.get("access_error"):
        return {
            "status": "access_failed",
            "reason": raw_result["access_error"],
            "isRetryable": True  # 접근 오류는 재시도 가능
        }
    
    elif raw_result.get("document_empty"):
        return {
            "status": "empty_document",
            "reason": "문서가 비어있음",
            "isRetryable": False  # 빈 문서는 재시도 불필요
        }
    
    elif all(v is None for k, v in raw_result.items() if k != "invoice_number"):
        return {
            "status": "extraction_failed",
            "partial_data": raw_result,
            "reason": "필드 대부분 추출 실패",
            "isRetryable": True
        }
    
    else:
        return {
            "status": "success",
            "data": raw_result
        }
```

---

## 멀티패스 추출

```python
def multipass_extraction(documents: list[str]) -> dict:
    """멀티패스: 로컬 추출 → 크로스파일 통합"""
    
    # 1단계: 각 문서별 개별 추출
    individual_results = []
    for doc in documents:
        result = extract_with_validation(doc)
        individual_results.append(result)
    
    # 2단계: 크로스파일 통합 분석 (독립 인스턴스)
    integration_prompt = f"""
다음은 여러 인보이스에서 추출된 데이터입니다.
중복 인보이스, 불일치 데이터, 이상값을 식별하세요:

{json.dumps(individual_results, ensure_ascii=False)}

통합 분석:
1. 중복 인보이스 번호
2. 동일 벤더의 데이터 불일치
3. 비정상적인 금액 범위
"""
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        messages=[{"role": "user", "content": integration_prompt}]
    )
    
    return {
        "individual": individual_results,
        "cross_document_analysis": response.content[0].text
    }
```

---

## 시나리오 기반 예상 문제

### Q: 환각 방지

상황: 인보이스 추출 시스템에서 일부 문서에 세율 정보가 없는데, 시스템이 임의의 세율을 생성합니다.

가장 효과적인 해결책은?

A) 시스템 프롬프트에 "세율이 없으면 추측하지 말라"고 지시  
B) 세율 필드를 `{"type": ["number", "null"]}`로 설정하고 "찾을 수 없으면 null" 설명 추가  
C) 세율 필드를 required에서 제거  
D) 추출 후 항상 세율이 합리적인지 검증  

정답: B — nullable 타입 + 명확한 null 사용 지시 → 환각 방지. (A는 확률적 준수만, C는 스키마 불완전)

---

### Q: 재시도 유효성

상황: 데이터 추출 후 유효성 검사에서 날짜 형식 오류가 발견됩니다. 이 오류는 재시도로 수정 가능한가요?

올바른 분류는?

A) transient (재시도 가능) - 서버 오류  
B) validation (재시도 가능) - 입력 형식 수정 후 재시도  
C) business (재시도 불가) - 정책 위반  
D) permission (재시도 불가) - 접근 권한  

정답: B — validation 오류는 일반적으로 재시도 불가이지만, 여기서는 LLM이 형식을 수정하여 재시도 가능. 컨텍스트에 따라 판단 필요.

> 💡 시험 팁: 데이터 추출에서 유효성 오류는 피드백을 주고 재시도하는 것이 일반적 패턴

---

## 📝 챕터 요약

| 개념 | 핵심 내용 |
|------|---------|
| nullable 필드 | 없는 정보 → null (환각 방지) |
| tool_choice 강제 | 특정 추출 툴 반드시 사용 |
| 검증-재시도 루프 | 오류 피드백 포함 재시도 |
| 빈 결과 vs 접근 실패 | 명확히 구분하여 처리 |
| 멀티패스 | 개별 분석 → 크로스파일 통합 |
| Few-shot | 다양한 형식 처리 (2-4개 예시) |

---

> 🔗 다음 챕터: [샘플 문제 해설 Q1~Q4](23_sample_q1_4.md)
