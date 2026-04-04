# Chapter 18: 시나리오 2 — Claude Code로 코드 생성

> 📅 2026년 04월 05일 기준  
> 🎯 실제 시험 시나리오 2 해설

---

## 시나리오 개요

> 당신은 소프트웨어 개발 팀을 위해 Claude Code를 구성하고 있습니다.
> 팀은 React/TypeScript 프론트엔드와 Python 백엔드를 가진 대규모 모노레포를 관리합니다.
> 목표: Claude Code를 생산성 도구로 효과적으로 통합하고, 코드 품질을 유지하면서 개발 속도를 높이는 것

---

## 핵심 아키텍처 결정

### CLAUDE.md 계층 구조 설계

```
모노레포 구조:
/
├── .claude/
│   ├── CLAUDE.md          ← 프로젝트 전체 규칙 (팀 공유)
│   ├── commands/          ← 팀 공유 슬래시 커맨드
│   │   ├── review.md      ← /review 커맨드
│   │   ├── test.md        ← /test 커맨드
│   │   └── deploy.md      ← /deploy 커맨드
│   └── rules/
│       ├── frontend.yaml  ← React/TS 규칙
│       └── backend.yaml   ← Python 규칙
├── frontend/
│   ├── CLAUDE.md          ← 프론트엔드 전용 규칙
│   └── src/
│       └── components/
│           └── CLAUDE.md  ← 컴포넌트 레벨 규칙
├── backend/
│   └── CLAUDE.md          ← 백엔드 전용 규칙
└── ~/ (사용자 홈)
    └── .claude/
        ├── CLAUDE.md      ← 개인 설정 (팀 공유 ❌)
        └── commands/      ← 개인 커맨드
```

### 프로젝트 CLAUDE.md 설계

```markdown
# 프로젝트 가이드라인

## 기술 스택
- 프론트엔드: React 18, TypeScript 5.4, Vite
- 백엔드: Python 3.12, FastAPI, PostgreSQL
- 테스트: pytest (백엔드), Vitest + RTL (프론트엔드)

## 코드 규칙
- 모든 공개 API는 타입 힌트 포함
- 커밋 전 반드시 테스트 통과 확인
- PR 크기: 200줄 이하 권장

## 금지 사항
- console.log() 프로덕션 코드에 포함 금지
- TODO 주석 없이 미완성 코드 커밋 금지
- 하드코딩된 자격증명 금지

@backend/CLAUDE.md
@frontend/CLAUDE.md
```

### .claude/rules/ 설정

```yaml
# frontend.yaml
---
paths:
  - "frontend/**/*.tsx"
  - "frontend/**/*.ts"
  - "**/*.test.tsx"
---
React 컴포넌트 작성 규칙:
- Props 인터페이스 반드시 정의
- memo(), useCallback()은 실제 성능 이슈 있을 때만 사용
- 컴포넌트당 하나의 책임
```

```yaml
# backend.yaml
---
paths:
  - "backend/**/*.py"
  - "tests/**/*.py"
---
Python 코드 규칙:
- type hints 필수
- docstring: Google 스타일
- 예외 처리: 구체적인 예외 클래스 사용
```

### 팀 공유 커맨드 설계

```markdown
<!-- .claude/commands/review.md -->
---
description: "코드 리뷰 체크리스트 실행"
argument-hint: "PR 번호 또는 파일 경로 (선택)"
allowed-tools: Read, Grep, Glob
---

다음 체크리스트로 코드를 리뷰하세요:

## 보안 검토
- [ ] SQL 인젝션 취약점
- [ ] XSS 가능성
- [ ] 인증/인가 검사

## 코드 품질
- [ ] 함수 복잡도 (10 이하)
- [ ] 중복 코드
- [ ] 타입 안전성

## 테스트
- [ ] 새 코드에 대한 테스트 존재
- [ ] 엣지 케이스 처리

$ARGUMENTS
```

```markdown
<!-- .claude/commands/test.md -->
---
description: "테스트 실행 및 커버리지 확인"
context: fork
allowed-tools: Bash
---

1. 프론트엔드 테스트: `npm run test --coverage`
2. 백엔드 테스트: `pytest --cov=app tests/`
3. 커버리지 80% 미만 파일 식별
4. 실패 테스트 원인 분석

$ARGUMENTS
```

---

## Plan Mode vs Direct Execution 판단

### Plan Mode 사용 시나리오

```
✅ Plan Mode 필요:
- 마이크로서비스 분리 (45개+ 파일 영향)
- 인증 시스템 전면 교체
- 데이터베이스 스키마 마이그레이션
- API 버전 업그레이드 (v1 → v2)

❌ Plan Mode 불필요:
- 단일 파일 버그 수정
- 명확한 스택 트레이스가 있는 오류
- 새 유틸리티 함수 추가
- 로컬라이제이션 문자열 업데이트
```

### Plan Mode 결정 트리

```
변경이 필요한가?
    ↓
여러 파일에 걸쳐 있는가?
    ├─ 아니오 → Direct Execution
    └─ 예 → 아키텍처적 결정이 필요한가?
                ├─ 아니오 → Direct Execution
                └─ 예 → Plan Mode
```

---

## CI/CD 통합

### GitHub Actions 파이프라인

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review

on:
  pull_request:
    branches: [main, develop]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Claude Code Review
        run: |
          claude -p "
          PR의 변경된 파일을 검토하세요.
          보안 취약점, 코드 품질 문제, 성능 이슈를 식별하세요.
          각 문제를 파일:라인 형식으로 명확히 표시하세요.
          
          변경된 파일:
          $(git diff --name-only origin/main...HEAD)
          " \
          --output-format json \
          --max-tokens 4096
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### 비대화형 모드 활용

```bash
# CI/CD에서 Claude Code 사용
# -p (--print): 비대화형 모드, 한 번 실행 후 종료
# --output-format json: 기계 처리 가능한 JSON 출력

claude -p "코드 분석 작업" --output-format json | jq '.result'

# JSON 스키마 강제
claude -p "분석 수행" \
  --output-format json \
  --json-schema '{"type": "object", "properties": {"issues": {"type": "array"}}}'
```

---

## 세션 관리 전략

### 장기 세션 관리

```bash
# 이름 있는 세션 시작
claude --session-name "feature-auth-redesign"

# 다음 날 세션 재개
claude --resume "feature-auth-redesign"

# fork_session: 공통 기준점에서 독립 탐색
# 예: 동일한 버그에 대해 두 가지 해결책 탐색
```

### context: fork 사용

```markdown
<!-- .claude/commands/explore.md -->
---
description: "독립적인 코드 탐색"
context: fork    ← 격리된 서브에이전트로 실행
---

이 커맨드는 현재 세션에 영향 없이
독립적으로 코드를 분석합니다.
```

---

## 시나리오 기반 예상 문제

### Q: 디렉토리별 규칙 적용

상황: React 컴포넌트 디렉토리에만 적용되는 규칙을 설정하고 싶습니다.

최선의 방법은?

A) 프로젝트 CLAUDE.md에 모든 규칙 한꺼번에 작성  
B) `.claude/rules/` 디렉토리에 glob 패턴이 있는 YAML 파일 생성  
C) 각 개발자의 `~/.claude/CLAUDE.md`에 규칙 추가  
D) `config.json`의 rules 배열에 규칙 정의  

정답: B — `.claude/rules/`의 YAML 파일에 `paths` frontmatter로 glob 패턴 지정

---

### Q: Plan Mode 판단

상황: 모노리식 앱을 6개의 마이크로서비스로 분리하는 작업을 시작합니다.

올바른 접근은?

A) 바로 코드 변경 시작 (직접 실행)  
B) Plan Mode로 전체 마이그레이션 계획 수립 후 진행  
C) 각 서비스를 별도 세션에서 구현  
D) CI/CD 파이프라인을 먼저 설정  

정답: B — 45개+ 파일, 아키텍처 결정, 마이크로서비스 분리 → Plan Mode 사용 기준 충족

---

## 📝 챕터 요약

| 개념 | 핵심 내용 |
|------|---------|
| CLAUDE.md | 계층: 사용자 → 프로젝트 → 디렉토리 |
| .claude/rules/ | glob 패턴으로 특정 파일에만 규칙 적용 |
| 팀 커맨드 | .claude/commands/ (버전 관리, 공유) |
| Plan Mode | 대규모 아키텍처 변경에 사용 |
| -p 플래그 | CI/CD 비대화형 모드 필수 |
| context: fork | 격리된 독립 탐색 |

---

> 🔗 다음 챕터: [시나리오 3 — 멀티에이전트 연구 시스템](19_scenario3_multi_agent.md)
