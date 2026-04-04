# 09.3 .claude/rules/ 조건부 규칙

> 📅 2026년 04월 05일 기준  
> ⭐ 시험 빈출 — YAML frontmatter 형식 암기

---

## .claude/rules/ 개요

CLAUDE.md는 전체 프로젝트에 적용되지만,
`.claude/rules/`는 특정 파일 유형에만 조건부 적용됩니다.

---

## YAML 파일 형식

```yaml
# .claude/rules/frontend.yaml
---
paths:
  - "frontend/**/*.tsx"     ← glob 패턴
  - "frontend/**/*.ts"
  - "**/*.test.tsx"
---

React 컴포넌트 작성 규칙:
- Props 인터페이스를 항상 정의하세요
- useEffect 의존성 배열 빠뜨리지 마세요
- 컴포넌트당 하나의 책임만 가지세요
- memo(), useCallback()은 실제 성능 이슈 확인 후 사용
```

```yaml
# .claude/rules/backend.yaml
---
paths:
  - "backend/**/*.py"
  - "tests/**/*.py"
---

Python 코드 규칙:
- 모든 공개 함수에 type hints 필수
- docstring은 Google 스타일 사용
- 예외는 구체적인 예외 클래스 사용 (Exception 제외)
- f-string 사용 (% 포맷 금지)
```

```yaml
# .claude/rules/terraform.yaml
---
paths:
  - "terraform/**/*.tf"
  - "infrastructure/**/*"
---

Terraform 규칙:
- 모든 리소스에 태그 필수 (project, environment, owner)
- 하드코딩된 IP 금지 → 변수 사용
- plan 전 항상 fmt 실행
```

---

## CLAUDE.md vs .claude/rules/

| 항목 | CLAUDE.md | .claude/rules/ |
|------|----------|----------------|
| 적용 범위 | 전체 프로젝트 | 특정 파일 유형만 |
| 조건부 적용 | ❌ | ✅ (glob 패턴) |
| 형식 | Markdown | YAML + Markdown |
| 사용 목적 | 일반 가이드라인 | 파일 유형별 규칙 |

---

## Glob 패턴 예시

```
"**/*.test.tsx"        — 모든 React 테스트 파일
"src/**/*.py"          — src 아래 Python 파일
"terraform/**/*"       — terraform 폴더 전체
"*.md"                 — 루트 마크다운만
"docs/**/*.{md,mdx}"   — docs 아래 md/mdx
```

---

> 🔗 다음: [Chapter 10: 커스텀 커맨드와 Skills](../10_commands_skills.md)
