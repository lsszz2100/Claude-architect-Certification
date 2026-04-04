# 13.1 tool_use와 JSON 스키마

> 📅 2026년 04월 05일 기준  
> ⭐ **시험 핵심 — tool_use가 보장하는 것 vs 않는 것**

---

## tool_use로 구조화된 출력

```python
# tool_use 사용 시 JSON 구문 오류 API 수준에서 제거
response = client.messages.create(
    model="claude-sonnet-4-6",
    tools=[{
        "name": "extract_data",
        "description": "데이터 추출",
        "input_schema": {
            "type": "object",
            "properties": {
                "name": {"type": "string"},
                "amount": {"type": "number"},
                "date": {"type": ["string", "null"]}
            },
            "required": ["name", "amount", "date"]
        }
    }],
    tool_choice={"type": "tool", "name": "extract_data"},
    messages=[{"role": "user", "content": "..."}]
)

# 결과는 항상 올바른 JSON 구조
data = response.content[0].input
```

---

## tool_use가 보장하는 것

```
✅ JSON 구문 오류 없음
✅ 스키마에 정의된 필드 존재
✅ 필드 타입 (string, number, boolean, null 등)

❌ 보장 안 함:
- 값이 실제로 정확한가 (의미적 정확성)
- 숫자 합산이 맞는가
- 날짜가 실제로 유효한가
→ 이는 별도 검증 필요!
```

---

## 일반 텍스트 응답 vs tool_use

```python
# 일반 텍스트 응답 — JSON 추출 불안정
response_text = """
{
  "name": "홍길동",
  "amount": 150.00
  "date": null  ← 콤마 누락 — 파싱 오류!
}
"""

# tool_use — API 수준 보장
tool_result = {
    "name": "홍길동",
    "amount": 150.00,
    "date": None    ← 항상 올바른 형식
}
```

---

## 스키마 설계 팁

```python
# 선택적 필드는 nullable로
{
    "type": "object",
    "properties": {
        "required_field": {"type": "string"},
        "optional_field": {
            "type": ["number", "null"],   # ← nullable!
            "description": "없으면 null"
        }
    },
    "required": ["required_field", "optional_field"]
    # optional_field도 required에 포함하되 null 허용
}
```

---

> 🔗 다음: [13.2 스키마 설계 원칙](13_2_schema_design.md)
