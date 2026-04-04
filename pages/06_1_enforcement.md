# 06.1 프로그래밍적 강제 vs 프롬프트 지시

> 📅 2026년 04월 05일 기준  
> ⭐ 시험 핵심 개념

---

## 핵심 원칙

```
Critical 비즈니스 로직 → 프로그래밍적 강제 (결정론적)
스타일/선호도          → 프롬프트 지시 (확률적)
```

---

## 프롬프트 기반의 한계

```python
# ❌ 프롬프트로만 강제하는 방법 — 불충분
SYSTEM_PROMPT = """
중요: 반드시 get_customer를 먼저 호출한 후
lookup_order를 호출하세요. 절대로 순서를 바꾸지 마세요.
"""

# 결과: 12%의 케이스에서 여전히 순서 위반
# LLM은 확률적으로 동작하므로 100% 보장 불가
```

---

## 프로그래밍적 강제

```python
# ✅ 코드로 강제하는 방법 — 결정론적 보장
class WorkflowGate:
    def __init__(self):
        self.customer_verified = False
        self.customer_id = None
    
    def execute_tool(self, tool_name: str, tool_input: dict) -> dict:
        """프로그래밍적 게이트 적용"""
        
        # 고객 확인 없이 주문 조회/환불 차단
        if tool_name in ["lookup_order", "process_refund"]:
            if not self.customer_verified:
                return {
                    "isError": True,
                    "errorCategory": "validation",
                    "message": "고객 확인 필요. get_customer를 먼저 호출하세요.",
                    "isRetryable": False
                }
        
        # 실제 툴 실행
        if tool_name == "get_customer":
            result = self._get_customer(tool_input)
            if result.get("customer_id"):
                self.customer_verified = True
                self.customer_id = result["customer_id"]
            return result
        
        # ...
```

---

## 비교표

| 항목 | 프롬프트 지시 | 프로그래밍적 강제 |
|------|-------------|----------------|
| 신뢰성 | 확률적 (~88%) | 결정론적 (100%) |
| 구현 난이도 | 낮음 | 높음 |
| 유연성 | 높음 | 낮음 |
| 감사 가능성 | 어려움 | 쉬움 |
| 사용 용도 | 스타일, 형식 | 보안, 금융 로직 |

---

## 언제 무엇을 사용하는가?

```
프로그래밍적 강제 사용:
✅ 결제/환불 처리 순서
✅ 인증/인가 검사
✅ 금액 한도 초과 방지
✅ 규정 준수 필수 항목

프롬프트 지시 사용:
✅ 응답 언어 (한국어로)
✅ 어조 (친절하게)
✅ 형식 (마크다운 사용)
✅ 길이 (200자 이내)
```

---

> 🔗 다음: [06.2 Agent SDK Hooks](06_2_hooks.md)
