# 5.2 코디네이터와 서브에이전트

> 📅 2026년 04월 05일 기준  
> ⭐ **시험 빈출 주제**

---

## 역할 구분

### 코디네이터
- 전체 목표 이해
- 태스크 분해
- 서브에이전트 스폰 (Task 툴)
- 결과 통합
- 최종 응답 생성

### 서브에이전트
- 특정 도메인 전문 작업
- 독립적으로 실행
- 코디네이터 컨텍스트 자동 상속 ❌
- 결과를 코디네이터에 반환

---

## ⭐ 핵심: 컨텍스트 자동 상속 없음

```python
# ❌ 틀린 가정
coordinator_context = {
    "client": "삼성전자",
    "deadline": "2024-12-31",
    "budget": "1억원"
}

# 서브에이전트는 이 정보를 모릅니다!
subagent = spawn_subagent("시장 분석 수행")
# subagent는 client, deadline, budget을 모름

# ✅ 올바른 방법: 명시적 전달
subagent = spawn_subagent(f"""
클라이언트: {coordinator_context['client']}
마감: {coordinator_context['deadline']}
예산: {coordinator_context['budget']}

위 정보를 고려하여 시장 분석을 수행하세요.
""")
```

---

## 코디네이터 구현 핵심

```python
COORDINATOR_SYSTEM = """
당신은 연구 코디네이터입니다.

Task 도구를 사용하여 서브에이전트를 스폰하세요.
각 서브에이전트에게는 필요한 모든 컨텍스트를 명시적으로 전달하세요.
"""

# allowedTools에 "Task" 반드시 포함!
response = client.messages.create(
    model="claude-opus-4-6",
    tools=[{"type": "computer_use_20250124", "name": "Task"}],  # ← 필수!
    ...
)
```

---

> 🔗 다음: [5.3 컨텍스트 전달 전략](05_3_context_passing.md)
