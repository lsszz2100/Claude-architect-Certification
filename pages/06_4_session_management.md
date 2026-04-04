# 6.4 세션 관리와 fork_session

> 📅 2026년 04월 05일 기준

---

## 세션 관리 기본

```bash
# 이름 있는 세션 시작
claude --session-name "feature-auth-v2"

# 세션 재개
claude --resume "feature-auth-v2"

# 이전 세션 목록 확인
claude --list-sessions
```

---

## fork_session

```
fork_session: 공통 기준점에서 독립적인 탐색 경로 시작

공통 맥락 (base)
    │
    ├── 탐색 경로 A (독립적)
    │   방법론 1로 접근
    │
    └── 탐색 경로 B (독립적)
        방법론 2로 접근
```

### 사용 사례

```bash
# 동일한 버그에 대해 두 가지 해결책 탐색
# 경로 A: 캐싱 방식으로 해결
# 경로 B: 알고리즘 최적화로 해결

# A와 B의 결과를 비교하여 최선의 방법 선택
```

---

## context: fork in Skills

```markdown
<!-- .claude/commands/explore.md -->
---
description: "독립적인 코드 탐색"
context: fork    ← 현재 세션에 영향 없는 격리 실행
---

이 탐색은 현재 작업에 영향을 주지 않습니다.
자유롭게 실험해보세요.
```

---

## 세션 격리의 장점

```
장점:
✅ 실험적 변경이 메인 세션에 영향 없음
✅ 여러 접근법 동시 탐색
✅ 실패해도 메인 작업 보호
✅ 나중에 성공한 경로 병합 가능
```

---

## 장기 세션 관리 팁

```python
# 긴 세션에서 컨텍스트 관리
# 1. 진행 상황 스크래치패드 유지
# 2. 주요 결정사항 명시적 기록
# 3. 필요한 경우 세션 저장 후 재개

SCRATCHPAD = """
## 현재 진행 상황
완료: 인증 모듈 리팩토링
진행 중: 데이터베이스 마이그레이션
다음: API 엔드포인트 업데이트

## 주요 결정사항
- JWT 방식 유지 (OAuth 도입 비용 > 이점)
- PostgreSQL 14로 업그레이드
"""
```

---

> 🔗 다음: [Chapter 7: 효과적인 툴 설계](../07_tool_design.md)
