# 3.3 메시지 구조 이해하기

> 📅 2026년 04월 05일 기준

---

## 메시지 형식

```python
messages = [
    {"role": "user", "content": "첫 번째 질문"},
    {"role": "assistant", "content": "첫 번째 답변"},
    {"role": "user", "content": "두 번째 질문"},
    # 현재 요청은 마지막 user 메시지
]
```

규칙:
- 항상 `user`로 시작
- `user`와 `assistant`가 번갈아 등장
- 마지막은 `user`

---

## 멀티턴 대화 구현

```python
def multi_turn_chat():
    """멀티턴 대화 구현"""
    
    client = anthropic.Anthropic()
    messages = []
    
    while True:
        user_input = input("나: ")
        if user_input.lower() in ["종료", "exit", "quit"]:
            break
        
        messages.append({"role": "user", "content": user_input})
        
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            messages=messages
        )
        
        assistant_message = response.content[0].text
        messages.append({"role": "assistant", "content": assistant_message})
        
        print(f"ARIA: {assistant_message}")
```

---

## 콘텐츠 블록 (Content Blocks)

```python
# 멀티모달: 텍스트 + 이미지
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/jpeg",
                    "data": base64_image_data
                }
            },
            {
                "type": "text",
                "text": "이 이미지에서 무엇이 보이나요?"
            }
        ]
    }]
)
```

---

## tool_result 메시지

툴 실행 결과를 messages에 추가하는 방법:

```python
# 에이전틱 루프에서 사용
messages.append({
    "role": "user",
    "content": [
        {
            "type": "tool_result",
            "tool_use_id": block.id,  # 툴 호출 ID와 일치
            "content": json.dumps(result)
        }
    ]
})
```

---

> 🔗 다음: [Chapter 4: 에이전틱 시스템의 이해](../04_agentic_intro.md)
