# 10.3 context:fork와 격리 실행

> 📅 2026년 04월 05일 기준

---

## context:fork 동작 방식

```
일반 커맨드:
현재 세션 → /review 실행 → 세션 컨텍스트에 영향

context: fork:
현재 세션 → /review 실행 → 격리된 서브에이전트
                              (세션 영향 없음)
                              결과만 반환 ↗
```

---

## 격리가 필요한 이유

```python
# 문제 상황: 현재 세션에 영향을 주는 대규모 탐색
# /explore-entire-codebase 를 실행하면
# 현재 작업 컨텍스트가 오염됨

# 해결: context: fork로 격리
---
context: fork
---
코드베이스 전체를 탐색하고 아키텍처를 분석하세요.
현재 세션에는 결과 요약만 반환됩니다.
```

---

## fork_session vs context:fork

```
fork_session (CLI):
- 새 독립 탐색 경로 시작
- 나중에 병합 가능

context: fork (Skills frontmatter):
- 커맨드를 격리 실행
- 결과만 현재 세션으로 반환
- 세션 분기 없음
```

---

## 실전 예시: 테스트 생성

```markdown
<!-- .claude/commands/gen-tests.md -->
---
description: "테스트 자동 생성"
context: fork             ← 격리 실행
allowed-tools: Read, Write, Bash
argument-hint: "테스트할 파일 경로"
---

$ARGUMENTS 파일에 대한 포괄적인 테스트를 생성하세요.

이 작업은 격리된 환경에서 실행됩니다.
현재 편집 중인 코드에 영향을 주지 않습니다.
```

---

> 🔗 다음: [Chapter 11: Plan Mode와 개발 워크플로우](11_plan_mode.md)
