# 5.3 컨텍스트 전달 전략

> 📅 2026년 04월 05일 기준

---

## 컨텍스트 전달 패턴

### 패턴 1: 시스템 프롬프트에 포함

```python
def spawn_with_system_context(context: dict, task: str) -> str:
    """시스템 프롬프트로 컨텍스트 전달"""
    
    system = f"""당신은 전문 연구원입니다.

프로젝트 배경:
- 클라이언트: {context['client']}
- 연구 주제: {context['main_topic']}
- 담당 영역: {context['your_domain']}
- 마감: {context['deadline']}

이미 완료된 분석: {context.get('completed', '없음')}
"""
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        system=system,
        messages=[{"role": "user", "content": task}]
    )
    
    return response.content[0].text
```

### 패턴 2: 사용자 메시지에 포함

```python
def spawn_with_user_context(context: dict, task: str) -> str:
    """사용자 메시지로 컨텍스트 전달"""
    
    contextualized_task = f"""
[프로젝트 컨텍스트]
{json.dumps(context, ensure_ascii=False)}

[수행할 작업]
{task}
"""
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        messages=[{"role": "user", "content": contextualized_task}]
    )
    
    return response.content[0].text
```

---

## 컨텍스트 요약 전달

긴 컨텍스트를 서브에이전트에 전달할 때:

```python
def create_context_summary(full_context: dict) -> str:
    """서브에이전트를 위한 컨텍스트 요약 생성"""
    
    # 핵심 정보만 추출
    summary = {
        "project": full_context["project_name"],
        "client_requirements": full_context["requirements"][:3],  # 상위 3개만
        "constraints": full_context["constraints"],
        "your_role": full_context["subagent_role"]
    }
    
    return json.dumps(summary, ensure_ascii=False)
```

---

## ❌ 자주 하는 실수

```
실수: "서브에이전트가 알아서 이해하겠지"
결과: 관련 없는 분석, 클라이언트 요구사항 무시

실수: "글로벌 변수로 공유하면 되지"
결과: 경쟁 조건, 테스트 어려움

✅ 올바른 방법: 모든 필요 정보를 Task 호출 시 명시적으로 전달
```

---

> 🔗 다음: [5.4 병렬 서브에이전트 실행](05_4_parallel_execution.md)
