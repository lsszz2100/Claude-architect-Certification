# 5.4 병렬 서브에이전트 실행

> 📅 2026년 04월 05일 기준  
> ⭐ **시험 빈출 주제**

---

## 병렬 실행의 핵심

```
순차 실행:        병렬 실행:
A → B → C        A ┐
총 시간: 45분     B ├→ 완료
                  C ┘
                  총 시간: 15분
```

**병렬 실행 방법**: 코디네이터의 **단일 응답에서** 여러 Task를 동시에 호출

---

## ⭐ 핵심 구현 패턴

```python
# 코디네이터가 한 번의 응답에서 여러 Task 호출 = 병렬 실행
# 이것이 SDK에서 병렬 실행의 공식 패턴

COORDINATOR_PROMPT = """
다음 세 가지 분석을 동시에 수행하세요:

1. Task: 시장 규모와 성장률 분석
2. Task: 주요 경쟁사 분석
3. Task: 규제 환경 분석

모든 Task를 한 번에 호출하여 병렬로 실행하세요.
"""
```

---

## Python에서 병렬 실행 처리

```python
from concurrent.futures import ThreadPoolExecutor
import anthropic

def execute_subagent(task_description: str, context: dict) -> str:
    """개별 서브에이전트 실행"""
    client = anthropic.Anthropic()
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=4096,
        system=f"컨텍스트: {json.dumps(context)}",
        messages=[{"role": "user", "content": task_description}]
    )
    return response.content[0].text


def run_parallel_agents(tasks: list[dict]) -> list[str]:
    """여러 에이전트 병렬 실행"""
    
    with ThreadPoolExecutor(max_workers=len(tasks)) as executor:
        futures = [
            executor.submit(execute_subagent, task["description"], task["context"])
            for task in tasks
        ]
        
        results = [future.result() for future in futures]
    
    return results
```

---

## 병렬 실행 결과 처리

```python
# 병렬 실행 결과 수집 후 통합
tool_results = []
for future, tool_id in zip(futures, tool_ids):
    result = future.result()
    tool_results.append({
        "type": "tool_result",
        "tool_use_id": tool_id,
        "content": result
    })

# 모든 결과를 한 번에 messages에 추가
messages.append({"role": "user", "content": tool_results})
```

---

> 🔗 다음: [Chapter 6: 워크플로우 설계](../06_workflow_design.md)
