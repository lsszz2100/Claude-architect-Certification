# 9.2 CLAUDE.md 설정 계층 구조

> 📅 2026년 04월 05일 기준  
> ⭐ 시험 핵심 — 계층 우선순위 암기 필수

---

## CLAUDE.md 계층

```
우선순위:  낮음 ←─────────────────→ 높음

~/.claude/CLAUDE.md          ← 사용자 (팀 공유 ❌)
    ↓ (덮어쓰기)
.claude/CLAUDE.md            ← 프로젝트 (팀 공유 ✅)
    ↓ (덮어쓰기)
src/CLAUDE.md                ← 서브디렉토리
    ↓ (덮어쓰기)
src/components/CLAUDE.md     ← 더 구체적인 디렉토리 (가장 높음)
```

---

## 각 수준의 역할

### 사용자 수준 (~/.claude/CLAUDE.md)
```markdown
# 내 개인 설정

## 코드 스타일 선호도
- 변수명: camelCase
- 들여쓰기: 2 스페이스

## 개인 단축키 및 알림
```

팀 공유 안 됨 — 개인 기기에만 존재

### 프로젝트 수준 (.claude/CLAUDE.md)
```markdown
# 프로젝트 전체 가이드라인

## 기술 스택
- Python 3.12, FastAPI
- React 18, TypeScript

## 필수 규칙
- 모든 API에 타입 힌트 필수
- PR 200줄 이하

@backend/CLAUDE.md    ← 임포트
@frontend/CLAUDE.md   ← 임포트
```

팀 공유 — 버전 관리

---

## @import 문법

```markdown
# 루트 CLAUDE.md
@backend/CLAUDE.md
@frontend/CLAUDE.md
@docs/api-guidelines.md
```

서브 CLAUDE.md 파일을 루트에서 참조 가능

---

## 우선순위 실제 예시

```
루트 CLAUDE.md: "들여쓰기: 4 스페이스"
frontend/CLAUDE.md: "들여쓰기: 2 스페이스"

→ frontend/ 파일 작업 시: 2 스페이스 적용
→ backend/ 파일 작업 시: 4 스페이스 적용
```

---

> 🔗 다음: [9.3 .claude/rules/ 조건부 규칙](09_3_rules.md)
