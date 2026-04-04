# Chapter 7: 효과적인 툴 설계

> 📅 2026년 04월 05일 기준  
> 🎯 Domain 2: 18% — 툴 설명이 핵심


[← Chapter 6](06_workflow_design.md) | [목차](../TOC.md) | [Chapter 8: MCP →](08_mcp.md)

---

## 7.1 툴 설명(Description)의 중요성

> 🎯 시험 최빈출: "툴 설명이 LLM의 툴 선택에서 1차적 메커니즘"

### 왜 툴 설명이 중요한가?

Claude는 툴 설명(description)을 기반으로 어떤 툴을 언제 사용할지 결정합니다.

나쁜 툴 설명의 결과:
```
툴 A: description = "고객 정보를 가져옵니다"
툴 B: description = "주문 정보를 가져옵니다"

→ "주문 #12345 확인해줘" 라고 하면
   Claude가 툴 A를 잘못 선택할 수 있음!
```

### 좋은 툴 설명 작성법

```python
# ❌ 나쁜 툴 설명
tools = [
    {
        "name": "get_customer",
        "description": "고객 정보를 가져옵니다",  # 너무 짧고 모호
        "input_schema": {...}
    },
    {
        "name": "lookup_order", 
        "description": "주문 정보를 가져옵니다",  # 비슷한 설명
        "input_schema": {...}
    }
]

# ✅ 좋은 툴 설명
tools = [
    {
        "name": "get_customer",
        "description": """고객 계정 정보를 이메일 또는 전화번호로 검색합니다.
        
        사용 시점: 고객 신원 확인, 계정 상태 조회, 개인정보 확인 시
        
        입력 형식: 
        - email: 고객 이메일 주소 (예: user@example.com)
        - phone: 국제 형식 전화번호 (예: +82-10-1234-5678)
        
        반환값: customer_id, name, email, account_status, tier, created_at
        
        주의: 주문 정보 조회에는 lookup_order를 사용하세요.
              이 툴은 고객 계정 정보만 반환합니다.""",
        "input_schema": {
            "type": "object",
            "properties": {
                "email": {"type": "string", "description": "고객 이메일"},
                "phone": {"type": "string", "description": "전화번호"}
            }
        }
    },
    {
        "name": "lookup_order",
        "description": """주문 번호 또는 고객 ID로 주문 정보를 조회합니다.
        
        사용 시점: 특정 주문 상태 확인, 배송 추적, 주문 이력 조회 시
        
        입력 형식:
        - order_id: 주문 번호 (예: ORD-2025-001234)
        - customer_id: 고객 ID (get_customer 호출 후 획득)
        
        반환값: order_id, status, items, total_amount, shipping_info
        
        주의: 고객 계정 정보 조회에는 get_customer를 사용하세요.""",
        "input_schema": {
            "type": "object",
            "properties": {
                "order_id": {"type": "string"},
                "customer_id": {"type": "string"}
            }
        }
    }
]
```

---

## 7.2 툴 경계 설정과 분리

### 제네릭 툴 분리 원칙

```python
# ❌ 나쁜 예: 너무 제네릭한 툴
tools = [
    {
        "name": "analyze_document",
        "description": "문서를 분석합니다",
        # → Claude가 어떻게 분석할지 판단 불가
    }
]

# ✅ 좋은 예: 명확한 목적별 분리
tools = [
    {
        "name": "extract_data_points",
        "description": """구조화된 데이터 포인트를 문서에서 추출합니다.
        날짜, 숫자, 이름 등의 사실적 정보 추출에 사용합니다."""
    },
    {
        "name": "summarize_content",
        "description": """문서의 핵심 내용을 요약합니다.
        전체 맥락 파악이 필요한 경우에 사용합니다."""
    },
    {
        "name": "verify_claim_against_source",
        "description": """특정 주장이 소스 문서에 근거가 있는지 검증합니다.
        팩트체크, 출처 확인에 사용합니다."""
    }
]
```

### 툴 이름 변경으로 중복 해소

```python
# 문제: 비슷한 이름의 툴이 혼동을 야기
# analyze_content vs analyze_document

# 해결: 명확한 도메인별 이름으로 변경
# analyze_content → extract_web_results (웹 특화)
# analyze_document → extract_document_data (문서 특화)
```

---

## 7.3 구조화된 에러 응답 설계

> 🎯 시험 출제: isError 플래그, errorCategory

### 에러 분류 체계

| 에러 종류 | 예시 | 재시도 가능? |
|----------|------|------------|
| `transient` | 타임아웃, 서버 일시 장애 | ✅ 예 |
| `validation` | 잘못된 입력 형식 | ❌ 아니오 (수정 필요) |
| `business` | 정책 위반, 한도 초과 | ❌ 아니오 |
| `permission` | 권한 없음 | ❌ 아니오 |

### 구조화된 에러 응답 구현

```python
def process_refund(customer_id: str, amount: float, reason: str):
    """환불 처리 MCP 툴"""
    
    # 비즈니스 규칙 검증
    if amount > 1000:
        return {
            "isError": True,
            "errorCategory": "business",
            "isRetryable": False,
            "errorCode": "REFUND_LIMIT_EXCEEDED",
            "message": "자동 환불 한도($1,000)를 초과합니다.",
            "userMessage": "귀하의 환불 요청이 자동 처리 한도를 초과합니다. 전문 상담원을 연결해드리겠습니다.",
            "suggestedAction": "escalate_to_human",
            "metadata": {
                "requested_amount": amount,
                "auto_limit": 1000,
                "excess_amount": amount - 1000
            }
        }
    
    # 일시적 오류
    try:
        result = payment_gateway.process_refund(customer_id, amount)
        return {"success": True, "refund_id": result.id}
    
    except TimeoutError:
        return {
            "isError": True,
            "errorCategory": "transient",
            "isRetryable": True,
            "errorCode": "GATEWAY_TIMEOUT",
            "message": "결제 게이트웨이 연결 시간 초과",
            "retryAfterSeconds": 30
        }
    
    except PermissionError:
        return {
            "isError": True,
            "errorCategory": "permission",
            "isRetryable": False,
            "errorCode": "UNAUTHORIZED",
            "message": "해당 계정에 대한 환불 권한이 없습니다."
        }
```

### ⚠️ 빈 결과 vs 액세스 실패 구분

```python
# ❌ 잘못된 예: 빈 결과와 오류를 같이 처리
def search_orders(customer_id: str):
    try:
        orders = db.query(customer_id)
        return orders  # 주문 없어도 빈 리스트 반환
    except Exception:
        return []  # 오류도 빈 리스트로 반환 → 구분 불가!

# ✅ 올바른 예: 명확히 구분
def search_orders(customer_id: str):
    try:
        orders = db.query(customer_id)
        return {
            "success": True,
            "orders": orders,
            "count": len(orders),
            "message": f"{len(orders)}개의 주문을 찾았습니다" if orders else "주문 내역이 없습니다"
        }
    except DatabaseConnectionError:
        return {
            "isError": True,
            "errorCategory": "transient",
            "isRetryable": True,
            "message": "데이터베이스 연결 실패 — 주문 조회 불가"
        }
```

---

## 7.4 tool_choice 사용법

```python
# auto: Claude가 자율적으로 선택 (기본값)
tool_choice = {"type": "auto"}

# any: 반드시 툴 호출 (어떤 툴이든)
tool_choice = {"type": "any"}

# 강제 지정: 특정 툴 반드시 호출
tool_choice = {"type": "tool", "name": "extract_metadata"}
```

### 언제 무엇을 사용하나?

```python
# auto: 일반적인 경우
# "필요하면 툴 써도 되고 텍스트 응답해도 됨"

# any: 구조화된 출력이 필요한 경우
# "반드시 툴을 통해 구조화된 형태로 응답해야 함"

# 강제 지정: 특정 순서 보장이 필요한 경우
# "데이터 추출 먼저, 그 다음에 분석"
response = client.messages.create(
    ...,
    tool_choice={"type": "tool", "name": "extract_metadata"},
    # → Claude는 반드시 extract_metadata를 먼저 호출
)
```

---

## 📝 챕터 요약

- 툴 설명은 LLM의 툴 선택 핵심 메커니즘 → 상세하고 명확하게 작성
- 제네릭 툴 → 목적별 전용 툴로 분리
- 에러 분류: `transient`(재시도 가능) / `validation`/`business`/`permission`(불가)
- 빈 결과와 액세스 실패는 반드시 구분
- `tool_choice`: `"auto"` / `"any"` / 강제 지정

---

> 🔗 다음 챕터: [Model Context Protocol (MCP)](08_mcp.md)
