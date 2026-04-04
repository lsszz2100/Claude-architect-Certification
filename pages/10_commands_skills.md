# Chapter 10: 커스텀 커맨드와 Skills

> 📅 2026년 04월 05일 기준  
> 🎯 Domain 3 — 프로젝트 vs 사용자 범위 구분


[← Chapter 9](09_claude_code.md) | [목차](../TOC.md) | [Chapter 11: Plan Mode →](11_plan_mode.md)

---

## 10.1 슬래시 커맨드 만들기

> 🎯 시험 출제: .claude/commands/ vs ~/.claude/commands/

### 범위 구분

| 위치 | 공유 | 용도 |
|------|------|------|
| `.claude/commands/` | ✅ 팀 전체 (버전 관리) | 팀 표준 워크플로우 |
| `~/.claude/commands/` | ❌ 개인만 | 개인 선호 워크플로우 |

### 팀 공유 커맨드 만들기

```markdown
# .claude/commands/review.md

# 코드 리뷰 실행

다음 체크리스트에 따라 현재 변경사항을 검토해주세요:

## 보안 체크
- [ ] SQL 인젝션 취약점
- [ ] API 키 하드코딩 여부
- [ ] 입력값 검증 여부

## 성능 체크
- [ ] N+1 쿼리 문제
- [ ] 불필요한 루프
- [ ] 메모리 누수 가능성

## 코딩 컨벤션
- [ ] 타입 힌트 추가
- [ ] 독스트링 완성도
- [ ] 네이밍 컨벤션 준수

심각도 수준 정의:
- 🔴 Critical: 즉시 수정 필요
- 🟡 Warning: 수정 권장
- 🟢 Suggestion: 선택적 개선
```

```bash
# 팀원이 사용하는 방법
claude /review
```

### 개인 커맨드 예시

```markdown
# ~/.claude/commands/myreview.md

# 내 스타일 코드 리뷰

위의 팀 리뷰에 추가로:
- 한국어로 피드백 작성
- 실제 코드 예시 포함
- 최대 10개의 주요 이슈만 보고
```

---

## 10.2 Skills 시스템 심화

### Skills vs Commands 차이

| 특성 | Commands | Skills |
|------|----------|--------|
| 파일 위치 | `.claude/commands/*.md` | `.claude/skills/*.md` |
| 격리 실행 | ❌ 기본 없음 | ✅ `context: fork` 옵션 |
| 도구 제한 | ❌ 없음 | ✅ `allowed-tools` 옵션 |
| 인자 힌트 | ❌ 없음 | ✅ `argument-hint` 옵션 |
| 용도 | 단순 워크플로우 | 복잡한 격리 작업 |

### SKILL.md 구조

```markdown
---
name: codebase-analyzer
description: 코드베이스 전체 구조를 분석하고 요약합니다
context: fork
allowed-tools: Read, Grep, Glob
argument-hint: "분석할 디렉토리 경로 (기본: 현재 디렉토리)"
---

# 코드베이스 분석기

## 목표
{argument}의 코드베이스 구조를 분석하여 다음을 제공합니다:
1. 디렉토리 구조 요약
2. 주요 모듈 및 의존성
3. 핵심 클래스/함수 목록
4. 코드 품질 메트릭

## 분석 방법
1. Glob으로 파일 구조 파악
2. Grep으로 주요 클래스/함수 식별
3. Read로 핵심 파일 분석
4. 구조화된 보고서 생성

결과는 간결한 요약으로만 반환하세요 (컨텍스트 오염 방지).
```

---

## 10.3 context:fork와 격리 실행

> 🎯 시험 출제: context:fork의 목적

### context:fork가 필요한 경우

```
문제: 코드베이스 분석 스킬이 대량의 파일 내용을 
     메인 대화 컨텍스트에 쌓아버림
     → 이후 작업에서 컨텍스트가 꽉 참

해결: context: fork
     → 스킬이 별도 서브에이전트에서 실행
     → 상세 탐색 내용은 격리됨
     → 메인 컨텍스트에는 최종 요약만 반환
```

```yaml
# ✅ 격리가 필요한 스킬
---
context: fork  # 서브에이전트에서 격리 실행
allowed-tools: Read, Grep, Glob  # 읽기 전용 도구만
---

# ❌ 격리 불필요
---
# context: fork 없음 → 메인 컨텍스트에서 실행
---
```

### allowed-tools로 안전성 확보

```yaml
---
name: safe-file-creator
context: fork
allowed-tools: Write  # 파일 쓰기만 허용 (삭제, 실행 불가)
---

# 안전한 파일 생성기

{argument} 경로에 기본 프로젝트 구조를 생성합니다.
Write 도구만 사용하여 파일을 생성합니다.
기존 파일을 삭제하거나 명령을 실행하지 않습니다.
```

### argument-hint 활용

```yaml
---
name: deploy-checker
argument-hint: "배포 환경 (development/staging/production)"
---

# 배포 전 체크리스트

환경: {argument}

{argument}에 따른 배포 체크리스트를 실행합니다...
```

```bash
# argument-hint가 없으면 Claude가 인자를 요청
claude /deploy-checker
# → "배포 환경을 입력해주세요: development/staging/production"
```

---

## 10.4 Skills vs CLAUDE.md 선택 기준

```
CLAUDE.md = 항상 로드되는 보편적 기준
            (모든 작업에 적용되는 코딩 규칙, 보안 정책 등)

Skills = 필요할 때만 호출하는 특정 작업
         (코드베이스 분석, PR 리뷰, 배포 체크 등)
```

---

## 📝 챕터 요약

- 팀 커맨드: `.claude/commands/` (버전 관리 공유)
- 개인 커맨드: `~/.claude/commands/`
- `context: fork`: 스킬을 격리된 서브에이전트에서 실행
- `allowed-tools`: 스킬 실행 중 사용 가능한 도구 제한
- `argument-hint`: 인자 없이 호출 시 안내 메시지
- CLAUDE.md = 항상 적용 / Skills = 온디맨드

---

> 🔗 다음 챕터: [Plan Mode와 개발 워크플로우](11_plan_mode.md)
