# Chapter 19: 시나리오 3 — 멀티에이전트 연구 시스템

> 📅 2026년 04월 05일 기준  
> 🎯 실제 시험 시나리오 3 해설


[← Chapter 18](18_scenario2_code_generation.md) | [목차](../TOC.md) | [Chapter 20: 시나리오 4 →](20_scenario4_developer_productivity.md)

---

## 시나리오 개요

> 당신은 Claude Agent SDK를 사용하여 멀티에이전트 연구 시스템을 구축하고 있습니다.
> 시스템은 복잡한 비즈니스 주제를 분석하고 심층 보고서를 생성합니다.
> 여러 서브에이전트가 병렬로 연구를 수행하고, 코디네이터가 통합합니다.

---

## 아키텍처 설계

### Hub-and-Spoke 구조

```
                    ┌─────────────────┐
                    │  코디네이터 에이전트  │
                    │  (연구 주제 분해)   │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
    ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
    │ 서브에이전트 A  │ │ 서브에이전트 B  │ │ 서브에이전트 C  │
    │  시장 분석    │ │  경쟁사 분석   │ │  규제 환경    │
    └──────────────┘ └──────────────┘ └──────────────┘
              │              │              │
              └──────────────┼──────────────┘
                             ▼
                    ┌─────────────────┐
                    │  코디네이터 에이전트  │
                    │  (결과 통합 + 보고서) │
                    └─────────────────┘
```

### 코디네이터 구현

```python
import anthropic
import json
from concurrent.futures import ThreadPoolExecutor

client = anthropic.Anthropic()

COORDINATOR_PROMPT = """당신은 연구 코디네이터입니다.
복잡한 비즈니스 주제를 받아 여러 서브에이전트에게 분배하고,
결과를 통합하여 종합 보고서를 생성합니다.

Task 도구를 사용하여 서브에이전트를 스폰하세요.
각 서브에이전트는 독립적인 연구 영역을 담당합니다.
"""

def run_coordinator(research_topic: str) -> str:
    """코디네이터 에이전트 실행"""
    
    messages = [{
        "role": "user",
        "content": f"다음 주제를 완전히 연구하세요: {research_topic}"
    }]
    
    # 코디네이터는 Task 도구를 포함해야 서브에이전트 스폰 가능
    tools = [
        {
            "type": "computer_use_20250124",
            "name": "Task",
            "description": "서브에이전트를 스폰하여 독립적인 연구 수행"
        }
    ]
    
    while True:
        response = client.messages.create(
            model="claude-opus-4-6",
            max_tokens=8096,
            system=COORDINATOR_PROMPT,
            tools=tools,
            messages=messages
        )
        
        if response.stop_reason == "end_turn":
            return response.content[0].text
        
        elif response.stop_reason == "tool_use":
            messages.append({"role": "assistant", "content": response.content})
            
            # 병렬 서브에이전트 실행
            tool_calls = [b for b in response.content if b.type == "tool_use"]
            results = run_subagents_parallel(tool_calls)
            
            messages.append({"role": "user", "content": results})
```

### 병렬 서브에이전트 실행

```python
def run_subagent(task_description: str, context: dict) -> str:
    """개별 서브에이전트 실행
    
    중요: 서브에이전트는 코디네이터 컨텍스트를 자동으로 받지 않습니다!
    필요한 컨텍스트는 명시적으로 전달해야 합니다.
    """
    
    SUBAGENT_PROMPT = f"""당신은 전문 연구 에이전트입니다.
    
연구 주제: {context['main_topic']}
담당 영역: {context['assigned_domain']}
특별 지시사항: {context.get('special_instructions', '없음')}

위 영역에 대해 심층 분석을 수행하세요."""
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=4096,
        system=SUBAGENT_PROMPT,
        messages=[{"role": "user", "content": task_description}]
    )
    
    return response.content[0].text


def run_subagents_parallel(tool_calls: list) -> list:
    """모든 서브에이전트를 병렬로 실행"""
    
    results = []
    
    # ThreadPoolExecutor로 병렬 실행
    with ThreadPoolExecutor(max_workers=5) as executor:
        futures = {}
        
        for tool_call in tool_calls:
            if tool_call.name == "Task":
                task_input = tool_call.input
                future = executor.submit(
                    run_subagent,
                    task_input["description"],
                    task_input.get("context", {})
                )
                futures[future] = tool_call.id
        
        for future, tool_id in futures.items():
            result = future.result()
            results.append({
                "type": "tool_result",
                "tool_use_id": tool_id,
                "content": result
            })
    
    return results
```

---

## 태스크 분해 전략

### 범위 설정의 중요성

```
❌ 너무 좁은 분해 (범위 누락):
연구 주제: "전기차 시장"
서브에이전트 A: 테슬라만 분석
서브에이전트 B: 미국 시장만 분석
결과: 중국, 유럽, 신흥 플레이어 완전 누락

✅ 적절한 분해:
코디네이터 태스크: "전기차 시장 전체"를 어떻게 나눌지 먼저 판단
서브에이전트 A: 주요 제조사 (테슬라, BYD, 폭스바겐)
서브에이전트 B: 지역별 시장 (미국, 유럽, 아시아)
서브에이전트 C: 기술 트렌드 (배터리, 충전 인프라)
서브에이전트 D: 규제/정책 환경
```

### 동적 분해 vs 프롬프트 체이닝

```python
# 프롬프트 체이닝: 순차적, 고정 워크플로우
# 언제: 단계가 명확하고 각 단계가 이전 결과에 의존할 때

def prompt_chaining_approach(topic: str):
    # 1단계: 아웃라인 생성
    outline = generate_outline(topic)
    
    # 2단계: 각 섹션 작성 (아웃라인 필요)
    sections = [write_section(s, outline) for s in outline]
    
    # 3단계: 통합 및 편집 (모든 섹션 필요)
    report = integrate_and_edit(sections)
    return report


# 동적 분해: LLM이 판단, 적응적
# 언제: 주제 복잡도나 범위가 미리 알 수 없을 때

DYNAMIC_DECOMPOSITION_PROMPT = """
연구 주제: {topic}

1. 먼저 이 주제의 핵심 영역을 파악하세요
2. 각 영역에 대해 Task를 사용하여 전문 에이전트를 스폰하세요
3. 결과를 통합하여 종합 보고서를 작성하세요

판단: 이 주제에 필요한 분석 영역은 무엇인가요?
"""
```

---

## 컨텍스트 전달 패턴

### 명시적 컨텍스트 전달

```python
def spawn_subagent_with_context(coordinator_context: dict, task: str) -> str:
    """서브에이전트에게 필요한 컨텍스트를 명시적으로 전달"""
    
    # ❌ 틀린 가정: 서브에이전트가 코디네이터 컨텍스트를 알 것이다
    # ✅ 올바른 방법: 필요한 모든 정보를 명시적으로 전달
    
    context_summary = f"""
연구 프로젝트 배경:
- 주제: {coordinator_context['main_topic']}
- 클라이언트: {coordinator_context['client']}
- 마감: {coordinator_context['deadline']}
- 이미 완료된 분석: {coordinator_context['completed_analyses']}
- 당신의 담당 영역: {coordinator_context['your_domain']}
"""
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=4096,
        system=context_summary,
        messages=[{"role": "user", "content": task}]
    )
    
    return response.content[0].text
```

---

## 결과 통합 패턴

### 구조화된 결과 수집

```python
def integrate_research_results(subagent_results: list[dict]) -> str:
    """서브에이전트 결과를 통합하여 최종 보고서 생성"""
    
    results_formatted = "\n\n".join([
        f"## {r['domain']}\n{r['content']}"
        for r in subagent_results
    ])
    
    integration_prompt = f"""
다음은 각 전문 에이전트의 연구 결과입니다:

{results_formatted}

이 결과들을 통합하여:
1. 일관성 있는 최종 보고서 작성
2. 각 영역 간 연관성 분석
3. 종합 결론 및 추천사항 도출
"""
    
    response = client.messages.create(
        model="claude-opus-4-6",
        max_tokens=8096,
        messages=[{"role": "user", "content": integration_prompt}]
    )
    
    return response.content[0].text
```

---

## 시나리오 기반 예상 문제

### Q: 서브에이전트 컨텍스트 문제

상황: 코디네이터가 연구 주제와 클라이언트 정보를 가지고 있습니다. 서브에이전트를 스폰했을 때, 서브에이전트가 클라이언트 요구사항을 모르고 분석을 수행했습니다.

원인과 해결책은?

A) 코디네이터 모델을 더 강력한 것으로 교체  
B) 서브에이전트는 코디네이터 컨텍스트를 자동 상속하지 않으므로, Task 호출 시 필요한 컨텍스트를 명시적으로 전달  
C) 서브에이전트가 코디네이터 API를 직접 호출하도록 설정  
D) 공유 데이터베이스에 컨텍스트 저장  

정답: B — 서브에이전트 컨텍스트 자동 상속 없음, 명시적 전달 필수

---

### Q: 태스크 분해 범위 문제

상황: "글로벌 반도체 공급망 분석"을 요청받았습니다. 코디네이터가 3개 서브에이전트를 스폰했지만, 최종 보고서에서 동남아시아 공급망이 완전히 누락되었습니다.

가장 가능한 원인은?

A) 서브에이전트 수가 너무 적음  
B) 코디네이터의 태스크 분해 단계에서 전체 범위를 고려하지 않고 좁게 분해  
C) 모델 컨텍스트 윈도우 부족  
D) 병렬 실행 중 레이스 컨디션  

정답: B — 코디네이터 태스크 분해가 좁으면 범위 누락. 분해 단계에서 전체 범위 검토 필수

---

## 📝 챕터 요약

| 개념 | 핵심 내용 |
|------|---------|
| 서브에이전트 컨텍스트 | 자동 상속 없음, 명시적 전달 필수 |
| allowedTools | "Task" 포함해야 서브에이전트 스폰 가능 |
| 병렬 실행 | 한 응답에서 여러 Task 동시 호출 |
| 태스크 분해 | 전체 범위 고려 (좁으면 누락 발생) |
| 프롬프트 체이닝 | 순차, 고정 워크플로우 |
| 동적 분해 | LLM이 판단, 미지의 범위 |

---

> 🔗 다음 챕터: [시나리오 4 — 개발자 생산성 도구](20_scenario4_developer_productivity.md)
