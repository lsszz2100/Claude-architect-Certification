# 16.1 에스컬레이션 패턴 설계

> 📅 2026년 04월 05일 기준  
> ⭐ **시험 핵심 — 에스컬레이션 기준 암기**

---

## 에스컬레이션 기준 (암기 필수!)

### 즉시 에스컬레이션해야 하는 경우

```
✅ 에스컬레이션:
1. 고객이 명시적으로 요청:
   "사람과 통화하고 싶어요"
   "상담원 연결해 주세요"
   "매니저 바꿔주세요"

2. 정책 공백:
   시스템에 없는 예외 상황
   정책 문서가 다루지 않는 케이스

3. 진전 불가:
   동일 시도 3번 이상 실패
   기술적으로 처리 불가한 상황
```

### 에스컬레이션하면 안 되는 경우

```
❌ 에스컬레이션 금지:
- 고객이 화났을 때 (감정 기반)
- 에이전트 자신감이 낮을 때
- 단순히 복잡해 보일 때
- 처리에 시간이 걸릴 것 같을 때
```

---

## 잘못된 에스컬레이션 설계

```python
# ❌ 감정 기반 에스컬레이션
def should_escalate(message: str) -> bool:
    negative_words = ["화났", "불만", "실망", "짜증"]
    if any(word in message for word in negative_words):
        return True  # ← 이것이 잘못됨

# ❌ 자신감 기반
if confidence_score < 0.6:
    escalate()  # ← 이것도 잘못됨
```

---

## 올바른 에스컬레이션 설계

```python
# ✅ 명시적 기준 기반
ESCALATION_TRIGGERS = [
    "사람 연결",
    "상담원",
    "매니저",
    "직원",
    "human agent"
]

def should_escalate(message: str, context: dict) -> tuple[bool, str]:
    """명시적 기준으로 에스컬레이션 판단"""
    
    # 1. 고객 명시적 요청
    if any(trigger in message for trigger in ESCALATION_TRIGGERS):
        return True, "고객 명시적 요청"
    
    # 2. 정책 공백
    if context.get("policy_gap"):
        return True, "정책 공백"
    
    # 3. 진전 불가
    if context.get("retry_count", 0) >= 3:
        return True, "반복 시도 실패"
    
    return False, ""
```

---

## 구조화된 핸드오프

```python
# 에스컬레이션 시 항상 구조화된 요약 포함
def escalate(customer_id: str, reason: str, context: dict) -> dict:
    return {
        "customer_id": customer_id,
        "reason": reason,
        "summary": f"""
고객: {context['customer_name']}
주문: {context.get('order_id', '없음')}
문제: {context['issue_description']}
시도된 해결: {context.get('attempted_solutions', [])}
        """,
        "priority": determine_priority(context)
    }
```

---

> 🔗 다음: [16.2 에러 전파 전략](16_2_error_propagation.md)
