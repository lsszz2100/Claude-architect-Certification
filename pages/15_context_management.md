# Chapter 15: 컨텍스트 관리 전략

> 📅 2026년 04월 05일 기준  
> 🎯 Domain 5: 15% — Lost-in-the-Middle, 구조화된 사실 추출

---

## 15.1 컨텍스트 창의 한계와 대응

### 컨텍스트 누적 문제

```
초기 상태:
[시스템 프롬프트] [사용자 메시지]

여러 툴 호출 후:
[시스템 프롬프트] [사용자 메시지] 
[툴1 응답 - 40개 필드] [툴2 응답 - 40개 필드]
[툴3 응답 - 40개 필드] [툴4 응답 - 40개 필드]
...
→ 컨텍스트가 순식간에 가득 참!
```

### 툴 결과 트리밍 (Trimming)

```python
def trim_order_result(raw_order: dict) -> dict:
    """주문 조회 결과에서 필요한 필드만 추출"""
    
    # 원본: 40개 이상의 필드
    # 환불 처리에 필요한 5개만 유지
    return {
        "order_id": raw_order["order_id"],
        "status": raw_order["status"],
        "items": raw_order["items"],
        "total_amount": raw_order["total_amount"],
        "customer_id": raw_order["customer_id"]
        # 나머지 35개 필드는 제거
    }
```

---

## 15.2 Lost-in-the-Middle 효과

> 🎯 시험 최빈출 개념

### 현상 설명

```
긴 컨텍스트 처리 시 주의도 분포:

높음 ▲
     |█                              █
     |██                            ██
     |███                          ███
     |████                        ████
낮음 |─────████████████████████─────
     시작                         끝
     ← 잘 처리됨  중간 (누락 위험)  잘 처리됨 →
```

### 대응 전략

```python
def structure_for_attention(data: dict) -> str:
    """중요 정보를 앞뒤에 배치하여 주의도 최적화"""
    
    return f"""
## ⚠️ 핵심 사실 (반드시 참조)
고객 ID: {data['customer_id']}
주문 ID: {data['order_id']}
환불 금액: {data['refund_amount']}
기한: {data['deadline']}

## 상세 분석 결과
{data['detailed_analysis']}

## ✅ 요약 및 권장 조치
{data['summary']}
중요: 위의 핵심 사실과 일치하는지 확인하세요.
"""
```

---

## 15.3 Progressive Summarization 주의점

### 위험한 요약

```python
# ❌ 잘못된 요약 (수치 정보 손실)
wrong_summary = """
고객이 여러 주문에 대해 문제를 제기했습니다.
환불 처리가 필요한 상황입니다.
"""
# → "여러 주문", "어느 정도의 환불" → 실제 처리 불가!

# ✅ 올바른 요약 (수치 정보 보존)
correct_summary = """
고객 (ID: CUST-2024-789):
- 주문 #ORD-001: $45.99 환불 요청 (손상 제품)
- 주문 #ORD-002: $120.00 부분 환불 (배송 지연 보상)
- 총 환불 예정: $165.99
- 고객 요구: 영업일 기준 3일 이내 처리
"""
```

### 구조화된 사실 블록 패턴

```python
CASE_FACTS_TEMPLATE = """
## CASE FACTS (항상 참조)
고객 ID: {customer_id}
이름: {customer_name}
연락처: {contact}

주문 이슈:
{issues}

금액 정보:
{amounts}

기한 및 약속:
{deadlines}

현재 상태: {current_status}
"""

def create_case_facts(customer_data: dict) -> str:
    """각 프롬프트에 포함할 불변 사실 블록"""
    return CASE_FACTS_TEMPLATE.format(**customer_data)

# 매 API 호출마다 포함
def process_customer_request(customer_data: dict, user_message: str):
    case_facts = create_case_facts(customer_data)
    
    return client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        system=f"""
당신은 고객 지원 에이전트입니다.

{case_facts}

위의 CASE FACTS를 항상 참조하여 정확한 정보를 제공하세요.
""",
        messages=[{"role": "user", "content": user_message}]
    )
```

---

## 📝 챕터 요약

- 툴 결과 트리밍: 불필요한 필드 제거로 컨텍스트 절약
- Lost-in-the-Middle: 중간 정보가 누락될 수 있음 → 중요 정보는 앞뒤에 배치
- 요약 시 수치(금액, 날짜, 주문번호) 반드시 보존
- 구조화된 사실 블록: 매 프롬프트에 핵심 사실 포함

---

> 🔗 다음 챕터: [에스컬레이션과 신뢰성](16_escalation_reliability.md)
