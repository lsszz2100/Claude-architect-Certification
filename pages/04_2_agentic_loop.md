# 4.2 에이전틱 루프의 핵심 원리

> 📅 2026년 04월 05일 기준

---

## 에이전틱 루프 흐름

```
사용자 요청
    ↓
Claude 응답
    ↓
stop_reason 확인
    ├─ "end_turn" → 루프 종료, 응답 반환
    └─ "tool_use" → 툴 실행
                        ↓
                    결과를 messages에 추가
                        ↓
                    Claude 재호출
                        ↓
                    (반복)
```

---

## 완전한 에이전틱 루프 구현

```python
import anthropic
import json

client = anthropic.Anthropic()

def run_agent(user_message: str, tools: list) -> str:
    """기본 에이전틱 루프"""
    
    messages = [{"role": "user", "content": user_message}]
    
    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=tools,
            messages=messages
        )
        
        # ✅ 올바른 종료 조건
        if response.stop_reason == "end_turn":
            # 텍스트 응답 추출
            for block in response.content:
                if hasattr(block, "text"):
                    return block.text
            return "완료"
        
        elif response.stop_reason == "tool_use":
            # 어시스턴트 메시지 저장
            messages.append({
                "role": "assistant",
                "content": response.content
            })
            
            # 툴 실행 및 결과 수집
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    result = execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": json.dumps(result)
                    })
            
            # 툴 결과를 messages에 추가
            messages.append({
                "role": "user",
                "content": tool_results
            })
        
        else:
            # 예상치 못한 stop_reason 처리
            break
    
    return "예기치 않은 종료"
```

---

## 안전장치 추가

```python
def run_agent_safe(user_message: str, tools: list, max_iterations: int = 50):
    """안전장치가 있는 에이전틱 루프"""
    
    messages = [{"role": "user", "content": user_message}]
    iteration = 0
    
    while iteration < max_iterations:  # 무한 루프 방지
        iteration += 1
        
        response = client.messages.create(...)
        
        if response.stop_reason == "end_turn":
            break  # ← 주요 종료 조건
        
        # 툴 처리...
    
    if iteration >= max_iterations:
        print(f"⚠️ 최대 반복 횟수({max_iterations}) 도달")
```

---

> 🔗 다음: [4.3 stop_reason 이해하기](04_3_stop_reason.md)
