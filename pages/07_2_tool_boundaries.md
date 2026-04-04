# 07.2 툴 경계 설정과 분리

> 📅 2026년 04월 05일 기준

---

## 툴 경계 설계 원칙

### 단일 책임 원칙

```python
# ❌ 너무 많은 책임
{
    "name": "customer_operations",
    "description": "고객 조회, 주문 확인, 환불 처리, 에스컬레이션을 수행합니다"
}

# ✅ 단일 책임
[
    {"name": "get_customer", "description": "고객 조회만"},
    {"name": "lookup_order", "description": "주문 조회만"},
    {"name": "process_refund", "description": "환불만"},
    {"name": "escalate_to_human", "description": "에스컬레이션만"}
]
```

---

## 툴 수 최적화

```
적절한 툴 수: 에이전트당 4-8개

너무 많은 툴 (18개+):
- 모델이 어떤 툴을 써야 할지 혼동
- 선택 신뢰도 저하
- 엉뚱한 툴 호출 빈번

해결책:
- 에이전트 분리 (각 에이전트 4-5개 툴)
- 관련 기능끼리 묶어서 전문화
```

---

## 툴 통합 vs 분리

```python
# 통합이 좋은 경우: 항상 같이 사용되는 기능
# ❌ 분리 과도: 같이 쓰이는 기능을 굳이 분리
{"name": "get_customer_email"}
{"name": "get_customer_phone"}
# → 그냥 get_customer로 합치자

# 분리가 좋은 경우: 독립적으로 사용되는 기능
# ✅ 올바른 분리
{"name": "get_customer"}    # 고객 정보
{"name": "lookup_order"}    # 주문 정보 (독립적 사용 가능)
```

---

## 읽기 vs 쓰기 분리

```python
# 읽기 툴 (안전, 언제나 허용)
read_tools = ["get_customer", "lookup_order", "search_products"]

# 쓰기 툴 (주의, 추가 검증 필요)
write_tools = ["process_refund", "update_account", "create_order"]

# PreToolUse로 쓰기 전 검증 강제
def pre_hook(tool_name, tool_input):
    if tool_name in write_tools:
        return verify_authorization(tool_input)
```

---

> 🔗 다음: [07.3 구조화된 에러 응답 설계](07_3_error_responses.md)
