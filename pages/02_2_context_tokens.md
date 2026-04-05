# 02.2 컨텍스트 윈도우와 토큰 이해

> 📅 2026년 04월 05일 기준

---

## 컨텍스트 윈도우란?

```
모델이 한 번에 처리할 수 있는 최대 토큰 수

┌──────────────────────────────────────────┐
│         컨텍스트 윈도우 (1M 토큰)          │
│  시스템   │   대화 히스토리   │  현재 요청  │
│  프롬프트 │  (이전 메시지들)  │            │
└──────────────────────────────────────────┘
```

---

## 토큰 계산

```python
import anthropic

client = anthropic.Anthropic()

# 토큰 수 미리 확인
response = client.messages.count_tokens(
    model="claude-sonnet-4-6",
    messages=[{
        "role": "user",
        "content": "이 텍스트의 토큰 수를 알고 싶어요"
    }]
)

print(f"토큰 수: {response.input_tokens}")
```

---

## Lost-in-the-Middle 효과

핵심 개념 (시험 자주 출제):

```
컨텍스트 위치에 따른 정보 처리 품질:

처음 ████████████  ← 높음
중간 ████          ← 낮음 (놓치기 쉬움)
끝   ████████████  ← 높음

→ 중요 정보는 앞 또는 끝에 배치하세요!
```

### 실전 적용

```python
SYSTEM_PROMPT = """
[중요 규칙 - 항상 참조]  ← 앞에 배치

{긴 배경 정보 및 예시들}  ← 중간 (덜 중요)

[핵심 제약사항 요약]      ← 끝에 반복
"""
```

---

## 컨텍스트 한계 관리

```python
def trim_context(messages: list, max_tokens: int = 900_000) -> list:
    """컨텍스트 한계 도달 시 트리밍"""
    
    total = sum(estimate_tokens(m) for m in messages)
    
    while total > max_tokens and len(messages) > 1:
        # 오래된 메시지부터 제거 (시스템 메시지 유지)
        messages.pop(1)  # 두 번째 메시지부터 제거
        total = sum(estimate_tokens(m) for m in messages)
    
    return messages
```

---

> 🔗 다음: [Chapter 03: Claude API 첫걸음](03_api_basics.md)
