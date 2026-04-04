# Chapter 03: Claude API 첫걸음

> 📅 2026년 04월 05일 기준  
> 💻 실습 중심 챕터


[← Chapter 02](02_claude_models.md) | [목차](../TOC.md) | [Chapter 04: 에이전틱 시스템 →](04_agentic_intro.md)

---

## 03.1 API 키 발급과 환경 설정

### Step 1: Anthropic Console 접속

1. [https://console.anthropic.com](https://console.anthropic.com) 접속
2. 계정 생성 또는 로그인
3. "API Keys" 메뉴에서 새 키 생성

### Step 2: SDK 설치

```bash
# Python
pip install anthropic

# Node.js
npm install @anthropic-ai/sdk
```

### Step 3: 환경 변수 설정

```bash
# Linux/Mac
export ANTHROPIC_API_KEY="sk-ant-..."

# Windows PowerShell
$env:ANTHROPIC_API_KEY="sk-ant-..."

# .env 파일 (python-dotenv 사용)
ANTHROPIC_API_KEY=sk-ant-...
```

> ⚠️ 절대 API 키를 코드에 직접 하드코딩하지 마세요!  
> Git에 커밋되면 공개될 수 있습니다.

---

## 03.2 첫 번째 API 호출

### Python으로 시작하기

```python
import anthropic

# 클라이언트 초기화
client = anthropic.Anthropic()
# ANTHROPIC_API_KEY 환경 변수를 자동으로 사용

# 메시지 전송
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": "안녕하세요! 저는 Claude Certified Architect 자격증을 준비하고 있습니다. 간단히 소개해 주세요."
        }
    ]
)

# 응답 출력
print(message.content[0].text)
```

### 응답 구조 이해

```python
# message 객체의 주요 필드
print(message.id)           # 메시지 고유 ID
print(message.type)         # "message"
print(message.role)         # "assistant"
print(message.content)      # 응답 내용 리스트
print(message.model)        # 사용된 모델
print(message.stop_reason)  # "end_turn", "tool_use" 등
print(message.usage)        # 토큰 사용량

# 토큰 사용량 확인
print(f"입력 토큰: {message.usage.input_tokens}")
print(f"출력 토큰: {message.usage.output_tokens}")
```

---

## 03.3 메시지 구조 이해하기

### 기본 메시지 구조

```python
messages = [
    {
        "role": "user",      # 사용자 메시지
        "content": "질문 내용"
    },
    {
        "role": "assistant",  # AI 응답 (이전 대화 유지 시)
        "content": "응답 내용"
    },
    {
        "role": "user",
        "content": "후속 질문"
    }
]
```

### System Prompt 활용

```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system="""당신은 ARIA(Autonomous Research & Intelligence Assistant)입니다.
    사용자의 업무를 효율적으로 지원하는 전문 AI 비서입니다.
    항상 정확하고 구조화된 응답을 제공합니다.""",
    messages=[
        {"role": "user", "content": "오늘 할 일 목록을 정리해줘."}
    ]
)
```

### 다중 턴 대화 구현

```python
# ARIA 비서 기본 구현
def aria_chat():
    client = anthropic.Anthropic()
    conversation_history = []
    
    system_prompt = """당신은 ARIA입니다. 업무 자동화를 돕는 전문 AI 비서입니다."""
    
    print("ARIA: 안녕하세요! 무엇을 도와드릴까요?")
    
    while True:
        user_input = input("You: ")
        if user_input.lower() == "exit":
            break
        
        # 대화 히스토리에 추가
        conversation_history.append({
            "role": "user",
            "content": user_input
        })
        
        # API 호출
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            system=system_prompt,
            messages=conversation_history
        )
        
        assistant_message = response.content[0].text
        
        # 히스토리에 응답 추가
        conversation_history.append({
            "role": "assistant",
            "content": assistant_message
        })
        
        print(f"ARIA: {assistant_message}")

# 실행
aria_chat()
```

---

## 03.4 API 파라미터

| 파라미터 | 필수 | 설명 | 예시 |
|---------|------|------|------|
| `model` | ✅ | 사용할 Claude 모델 | `"claude-sonnet-4-6"` |
| `max_tokens` | ✅ | 최대 출력 토큰 수 | `1024` |
| `messages` | ✅ | 대화 히스토리 | `[{"role": "user", ...}]` |
| `system` | ❌ | 시스템 프롬프트 | `"당신은 전문 비서입니다"` |
| `temperature` | ❌ | 창의성 (0~1) | `0.7` |
| `tools` | ❌ | 사용 가능한 툴 목록 | `[{...}]` |
| `tool_choice` | ❌ | 툴 선택 방식 | `"auto"` |

### temperature 가이드

```
temperature = 0.0  → 결정론적, 항상 같은 답변 (분류, 추출에 적합)
temperature = 0.3  → 일관성 높음 (코드 생성, 분석에 적합)
temperature = 0.7  → 균형 (일반적인 작업)
temperature = 1.0  → 창의적, 다양한 답변 (브레인스토밍에 적합)
```

---

## 📝 챕터 요약

- API 키는 환경 변수로 관리 (절대 하드코딩 금지)
- `messages` 배열로 대화 히스토리 관리
- `stop_reason`: `"end_turn"` (정상 완료) / `"tool_use"` (툴 호출 필요)
- `temperature 0`: 일관된 결과 / `temperature 높을수록`: 창의적

---

> 🔗 다음 챕터: [에이전틱 시스템의 이해](04_agentic_intro.md)
