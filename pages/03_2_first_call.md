# 3.2 첫 번째 API 호출

> 📅 2026년 04월 05일 기준

---

## 가장 간단한 API 호출

```python
import anthropic

client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "안녕하세요! 자기소개를 해주세요."}
    ]
)

print(message.content[0].text)
```

---

## 응답 구조 이해

```python
# message 객체의 주요 속성
print(message.id)              # msg_01XFDUDYJgAACzvnptvVoYEL
print(message.model)           # claude-sonnet-4-6
print(message.stop_reason)     # "end_turn"
print(message.usage.input_tokens)   # 사용된 입력 토큰
print(message.usage.output_tokens)  # 사용된 출력 토큰
print(message.content[0].text)      # 실제 응답 텍스트
```

---

## 시스템 프롬프트 추가

```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system="당신은 친절한 고객 지원 에이전트입니다. 항상 한국어로 답변하세요.",
    messages=[
        {"role": "user", "content": "주문을 취소하고 싶어요."}
    ]
)
```

---

## 스트리밍 응답

```python
# 긴 응답을 실시간으로 처리
with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "긴 보고서를 써주세요."}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

---

## ARIA 실습: 첫 번째 자동화 비서

```python
import anthropic

client = anthropic.Anthropic()

def ask_aria(question: str) -> str:
    """ARIA에게 질문하기"""
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        system="""당신은 ARIA(AI Research & Intelligent Assistant)입니다.
업무 자동화를 돕는 친절하고 효율적인 비서입니다.""",
        messages=[{"role": "user", "content": question}]
    )
    
    return response.content[0].text

# 테스트
print(ask_aria("오늘 할 일 목록을 정리하는 방법을 알려주세요."))
```

---

> 🔗 다음: [3.3 메시지 구조 이해하기](03_3_message_structure.md)
