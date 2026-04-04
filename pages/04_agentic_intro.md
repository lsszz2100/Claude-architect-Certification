# Chapter 04: 에이전틱 시스템의 이해

> 📅 2026년 04월 05일 기준  
> 🎯 Domain 1: 27% 비중 — 가장 중요한 챕터


[← Chapter 03](03_api_basics.md) | [목차](../TOC.md) | [Chapter 05: 멀티에이전트 시스템 →](05_multi_agent.md)

---

## 04.1 에이전트란 무엇인가?

### 챗봇 vs 에이전트

```
챗봇 = 질문 → 답변 (단방향, 수동적)

에이전트 = 목표 → 계획 → 도구 사용 → 결과 확인 → 반복
         (자율적, 능동적, 멀티스텝)
```

### 에이전트가 할 수 있는 것

ARIA 비서 시나리오:
```
사용자: "이번 주 모든 고객 불만 이메일을 분석하고 
         카테고리별로 정리해서 대응 방안까지 제시해줘"

에이전트 처리 과정:
1. 이메일 서버에서 불만 이메일 수집 (툴 사용)
2. 각 이메일 감정 분석 및 카테고리 분류 (추론)
3. 통계 집계 (툴 사용)
4. 카테고리별 대응 방안 생성 (추론)
5. 보고서 작성 및 전달 (툴 사용)
```

이 모든 과정을 사람의 개입 없이 자동으로 처리합니다.

---

## 04.2 에이전틱 루프의 핵심 원리

> 🎯 시험 최빈출 포인트!

### 에이전틱 루프 작동 방식

```
┌─────────────────────────────────────────┐
│           에이전틱 루프                   │
│                                          │
│  1. 사용자 요청 전송                      │
│         ↓                               │
│  2. Claude 응답 수신                     │
│         ↓                               │
│  3. stop_reason 확인                    │
│    ┌────────────────────────────┐       │
│    │ "tool_use" → 툴 실행      │       │
│    │ "end_turn" → 루프 종료   │       │
│    └────────────────────────────┘       │
│         ↓ (tool_use인 경우)             │
│  4. 툴 실행 및 결과 수집                 │
│         ↓                               │
│  5. 결과를 대화 히스토리에 추가           │
│         ↓                               │
│  6. 다시 Claude에 전송 (1번으로 돌아감) │
└─────────────────────────────────────────┘
```

---

## 04.3 stop_reason 이해하기

> 🎯 시험 직출 — 반드시 암기

### stop_reason 종류

| stop_reason | 의미 | 다음 행동 |
|-------------|------|----------|
| `"end_turn"` | 모델이 응답 완료 | 루프 종료, 최종 답변 반환 |
| `"tool_use"` | 툴 호출 요청 | 툴 실행 후 결과를 히스토리에 추가 |
| `"max_tokens"` | 최대 토큰 도달 | max_tokens 증가 후 재시도 |
| `"stop_sequence"` | 지정된 중단 시퀀스 도달 | 설계에 따라 처리 |

### 올바른 에이전틱 루프 구현

```python
import anthropic
import json

client = anthropic.Anthropic()

def run_agent(tools, system_prompt, user_message):
    """올바른 에이전틱 루프 구현"""
    
    messages = [{"role": "user", "content": user_message}]
    
    while True:
        # Claude에 요청
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            system=system_prompt,
            tools=tools,
            messages=messages
        )
        
        # ✅ 올바른 종료 조건: stop_reason으로 판단
        if response.stop_reason == "end_turn":
            # 텍스트 응답 추출
            for block in response.content:
                if hasattr(block, 'text'):
                    return block.text
            break
        
        elif response.stop_reason == "tool_use":
            # 응답을 히스토리에 추가
            messages.append({
                "role": "assistant",
                "content": response.content
            })
            
            # 모든 툴 호출 처리
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    # 툴 실행
                    result = execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": json.dumps(result)
                    })
            
            # 툴 결과를 히스토리에 추가
            messages.append({
                "role": "user",
                "content": tool_results
            })
        
        else:
            # max_tokens 등 예외 처리
            break
    
    return None


def execute_tool(tool_name: str, tool_input: dict):
    """툴 실행 함수 (실제 구현 필요)"""
    if tool_name == "search_emails":
        return search_emails(tool_input.get("query"))
    elif tool_name == "classify_text":
        return classify_text(tool_input.get("text"))
    # ... 기타 툴
    return {"error": f"Unknown tool: {tool_name}"}
```

### ⚠️ 자주 틀리는 안티패턴

```python
# ❌ 잘못된 방법 1: 텍스트 내용으로 루프 종료 판단
if "TASK COMPLETE" in response.content[0].text:  # 위험!
    break

# ❌ 잘못된 방법 2: 고정 횟수로만 제한
for i in range(10):  # stop_reason 무시
    ...

# ❌ 잘못된 방법 3: 텍스트 응답 존재로 완료 판단
if response.content[0].type == "text":  # tool_use와 text가 함께 올 수 있음
    break

# ✅ 올바른 방법: stop_reason으로만 판단
if response.stop_reason == "end_turn":
    break
```

---

## 에이전트 설계 원칙

### 1. 최소 권한 원칙
에이전트에게 꼭 필요한 툴만 제공합니다.

### 2. 투명성 원칙
모든 툴 호출과 결과를 로깅합니다.

### 3. 안전 게이트 원칙
중요한 작업(돈, 데이터 삭제 등)은 반드시 사람의 확인을 거칩니다.

---

## 📝 챕터 요약

- 에이전트 = 목표를 위해 자율적으로 툴을 사용하는 시스템
- 에이전틱 루프: Claude 요청 → stop_reason 확인 → 툴 실행 → 반복
- `stop_reason == "end_turn"` → 루프 종료
- `stop_reason == "tool_use"` → 툴 실행 후 결과 추가
- 절대 텍스트 내용이나 고정 반복 횟수로 루프를 제어하지 말 것

---

> 🔗 다음 챕터: [멀티에이전트 시스템 설계](05_multi_agent.md)
