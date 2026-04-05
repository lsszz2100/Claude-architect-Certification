# 07.4 tool_choice 사용법

> 📅 2026년 04월 05일 기준

---

## tool_choice 옵션

```python
# 1. auto: 자율 선택 (기본값)
response = client.messages.create(
    tool_choice={"type": "auto"},  # 또는 생략
    ...
)
# LLM이 필요하다고 판단할 때만 툴 호출

# 2. any: 반드시 어떤 툴이든 호출
response = client.messages.create(
    tool_choice={"type": "any"},
    ...
)
# LLM이 반드시 제공된 툴 중 하나를 호출

# 3. 특정 툴 강제
response = client.messages.create(
    tool_choice={"type": "tool", "name": "extract_invoice_data"},
    ...
)
# 반드시 extract_invoice_data 툴 호출
```

---

## 각 옵션 사용 시나리오

```
auto:
  일반 대화 에이전트
  툴이 필요할 때도 있고 없을 때도 있는 경우

any:
  반드시 구조화된 응답이 필요한 경우
  텍스트 응답을 원하지 않을 때

특정 툴 강제:
  데이터 추출 (특정 형식으로만 받고 싶을 때)
  tool_use로 JSON 구문 오류를 제거할 때
  extract_invoice_data처럼 명확한 스키마가 있을 때
```

---

## 실전 예시: 데이터 추출

```python
# 인보이스 데이터 추출 — 특정 툴 강제
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=2048,
    tools=[EXTRACTION_TOOL],
    tool_choice={"type": "tool", "name": "extract_invoice_data"},  # 강제!
    messages=[{
        "role": "user",
        "content": f"다음 인보이스에서 데이터를 추출하세요:\n{document}"
    }]
)

# 결과는 항상 tool_use 형식 (JSON 구문 오류 없음 보장)
for block in response.content:
    if block.type == "tool_use":
        extracted_data = block.input
```

---

## tool_choice와 JSON 품질

```
tool_choice + 스키마 정의:
→ API 수준에서 JSON 구문 오류 제거
→ 스키마에 정의된 필드 포함 보장
→ 의미적 정확성은 별도 검증 필요

✅ 보장: 구문 올바른 JSON
⚠️ 미보장: 값이 실제로 정확한지 (의미적 검증 필요)
```

---

> 🔗 다음: [Chapter 08: Model Context Protocol (MCP)](08_mcp.md)
