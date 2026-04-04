# Chapter 9: Claude Code 실전

> 📅 2026년 04월 05일 기준  
> 🎯 Domain 3: 20% — CLAUDE.md 계층이 핵심

---

## 9.1 Claude Code란?

Claude Code는 Anthropic이 제공하는 AI 코딩 어시스턴트 CLI입니다.

```bash
# 설치
npm install -g @anthropic-ai/claude-code

# 기본 사용
claude "이 함수의 버그를 찾아줘"

# 파일 지정
claude "payment.py 파일의 환불 로직을 검토해줘"
```

### Claude Code의 핵심 기능

| 기능 | 설명 |
|------|------|
| 코드 생성 | 자연어 설명으로 코드 작성 |
| 디버깅 | 버그 원인 분석 및 수정 |
| 리팩토링 | 코드 구조 개선 |
| 문서화 | 주석, README 작성 |
| 테스트 생성 | 단위/통합 테스트 자동 생성 |
| 코드 리뷰 | PR 리뷰 및 개선 제안 |

---

## 9.2 CLAUDE.md 설정 계층 구조

> 🎯 시험 최빈출: 3단계 계층 구조

### 계층 구조

```
~/.claude/CLAUDE.md          ← 사용자 수준 (개인 설정, 팀 공유 ❌)
    ↓ (낮은 우선순위)
.claude/CLAUDE.md            ← 프로젝트 수준 (팀 공유 ✅)
또는 CLAUDE.md (루트)
    ↓
src/payment/CLAUDE.md        ← 디렉토리 수준 (해당 디렉토리 전용)
```

### 각 수준별 적합한 내용

사용자 수준 (~/.claude/CLAUDE.md)
```markdown
# 개인 설정

## 선호하는 코딩 스타일
- 타입 힌트 항상 추가
- Google 스타일 독스트링 사용
- 변수명은 camelCase 선호

## 개인 워크플로우
- 커밋 전 항상 테스트 실행
- PR 설명에 "Why" 섹션 포함

주의: 이 설정은 팀원들에게 공유되지 않습니다.
```

프로젝트 수준 (.claude/CLAUDE.md)
```markdown
# ARIA 프로젝트 코딩 가이드

## 프로젝트 개요
ARIA는 업무 자동화를 위한 멀티에이전트 시스템입니다.

## 코딩 컨벤션
- Python 3.10+ 사용
- 모든 함수에 타입 힌트 필수
- 에러는 구조화된 딕셔너리로 반환

## 테스트 규칙
- 모든 툴 함수는 단위 테스트 필수
- pytest 사용, coverage 80% 이상 유지

## 보안 규칙
- API 키는 환경 변수로만 관리
- 로그에 개인정보 포함 금지
- SQL 쿼리는 파라미터화 필수

@import ./docs/api-conventions.md
@import ./docs/testing-standards.md
```

디렉토리 수준 (src/payment/CLAUDE.md)
```markdown
# 결제 모듈 특별 지침

이 디렉토리는 결제 처리 핵심 로직을 포함합니다.

## 보안 요구사항
- 모든 금액은 Decimal 타입 사용 (부동소수점 오류 방지)
- 거래 전 반드시 고객 인증 확인
- 환불 금액 검증 필수

## 에러 처리
- 모든 결제 실패는 structured_error 형식으로 반환
- 재시도 가능 여부를 isRetryable로 명시
```

### @import 문법

```markdown
# CLAUDE.md에서 외부 파일 참조
@import ./docs/api-conventions.md
@import ./docs/testing-standards.md
@import ./docs/security-guidelines.md
```

> 💡 큰 CLAUDE.md를 주제별 파일로 분리하고 @import로 조합하면 유지보수가 쉬워집니다.

---

## 9.3 .claude/rules/ 조건부 규칙

> 🎯 시험 출제: glob 패턴으로 파일별 규칙 적용

### 기본 구조

```yaml
# .claude/rules/terraform-conventions.md
---
name: Terraform 코딩 규칙
paths:
  - "terraform/**/*"
  - "infra/**/*.tf"
---

# Terraform 파일 작성 규칙

## 명명 규칙
- 리소스 이름: snake_case 사용
- 변수: 설명적인 이름 사용

## 보안 규칙
- 민감한 변수는 sensitive = true 설정
- 하드코딩된 시크릿 절대 금지
```

```yaml
# .claude/rules/testing-conventions.md
---
name: 테스트 파일 규칙
paths:
  - "**/*.test.tsx"
  - "**/*.test.ts"
  - "**/*.spec.py"
  - "tests/**/*"
---

# 테스트 작성 규칙

## 구조
- Arrange-Act-Assert 패턴 사용
- 테스트명: "것_해야_결과" 형식

## 범위
- Happy path + Edge cases 모두 커버
- 목킹은 외부 서비스에만 사용
```

### ⚠️ 왜 디렉토리별 CLAUDE.md보다 rules/가 유리한가?

```
CLAUDE.md (디렉토리별) 문제:
- 테스트 파일이 여러 디렉토리에 분산되어 있으면
  각 디렉토리마다 CLAUDE.md를 만들어야 함 → 유지보수 어려움

rules/ (glob 패턴) 장점:
- **/*.test.tsx 패턴 하나로
  모든 테스트 파일에 동일한 규칙 적용 가능!
```

---

## 📝 챕터 요약

- CLAUDE.md 계층: 사용자 → 프로젝트 → 디렉토리 (순서대로 적용)
- 사용자 수준 설정은 버전 관리되지 않아 팀원과 공유 불가
- `@import`로 대형 CLAUDE.md를 모듈화
- `.claude/rules/`의 glob 패턴으로 파일 유형별 조건부 규칙 적용
- 여러 디렉토리에 걸친 파일 유형(예: 테스트 파일)에는 rules/가 유리

---

> 🔗 다음 챕터: [커스텀 커맨드와 Skills](10_commands_skills.md)
