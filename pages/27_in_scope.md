# Chapter 27: 시험 범위 포함 주제

> 📅 2026년 04월 05일 기준  
> 🎯 시험 출제 범위 요약


[← Chapter 26](26_practice_questions.md) | [목차](../TOC.md) | [Chapter 28: 비출제 범위 →](28_out_of_scope.md)

---

## 시험 범위 개요

Claude Certified Architect – Foundations 시험은 다음 5개 도메인을 평가합니다.
각 도메인에서 확실히 알아야 할 핵심 주제를 정리했습니다.

---

## Domain 1: Agentic Architecture (27%)

### ✅ 반드시 알아야 할 주제

#### 에이전틱 루프
- `stop_reason` 기반 루프 제어
  - `"tool_use"` → 툴 실행 후 계속
  - `"end_turn"` → 루프 종료
  - `"max_tokens"` → 토큰 한계 도달
- 안티패턴: 텍스트 키워드, 고정 횟수만으로 종료

#### 멀티에이전트 시스템
- Hub-and-Spoke 아키텍처
- 서브에이전트 컨텍스트 자동 상속 없음
- `allowedTools`에 `"Task"` 포함 필요
- 병렬 실행: 단일 응답에서 여러 Task 동시 호출

#### 프로그래밍적 강제
- Critical 비즈니스 로직 = 코드로 강제
- 프롬프트 기반 = 확률적 (불충분)
- 게이트 패턴: `verified_customer_id` 확인 후 `process_refund` 허용

#### Hooks
- `PostToolUse`: 툴 결과 변환/정규화
- `PreToolUse`: 툴 호출 차단, 정책 강제

#### 세션 관리
- `--resume <session-name>`: 이름 있는 세션 재개
- `fork_session`: 공통 기준점에서 독립 탐색

#### 태스크 분해
- 프롬프트 체이닝: 순차, 각 단계가 이전 결과 의존
- 동적 분해: LLM이 판단, 미지의 범위 처리
- 좁은 분해의 위험: 범위 누락

---

## Domain 2: Tool Design & MCP (18%)

### ✅ 반드시 알아야 할 주제

#### 툴 설명 설계
- 설명 = LLM 툴 선택의 1차적 메커니즘
- 포함 내용: 사용 시점, 입력 형식, 예제, 엣지 케이스, 비슷한 툴과 구분
- 비슷한 툴은 설명에서 명확히 구분

#### 에러 응답 구조
| 필드 | 역할 |
|------|------|
| `isError` | 오류 여부 |
| `errorCategory` | transient/validation/business/permission |
| `isRetryable` | 재시도 가능 여부 |
| `message` | 사람이 읽을 수 있는 설명 |

- transient: 재시도 가능 (타임아웃, 서버 오류)
- validation/business/permission: 재시도 불가

#### tool_choice 옵션
- `"auto"`: 자율 선택 (기본값)
- `"any"`: 반드시 어떤 툴이든 사용
- `{"type": "tool", "name": "..."}`: 특정 툴 강제

#### MCP (Model Context Protocol)
- 표준 프로토콜: Claude와 외부 시스템 연결
- `.mcp.json` (프로젝트): 팀 공유, 버전 관리
- `~/.claude.json` (사용자): 개인 전용
- `${변수명}`: 환경 변수 참조

#### 내장 툴 선택 기준
| 툴 | 사용 시점 |
|----|----------|
| Grep | 파일 내용에서 패턴 검색 |
| Glob | 파일 경로 패턴으로 파일 찾기 |
| Read | 파일 전체 내용 읽기 |
| Edit | 유일한 텍스트 교체 |
| Write | 파일 전체 재작성 (Read 먼저!) |

---

## Domain 3: Claude Code (20%)

### ✅ 반드시 알아야 할 주제

#### CLAUDE.md 계층
```
우선순위 높음
~/.claude/CLAUDE.md          ← 사용자 (팀 공유 ❌)
.claude/CLAUDE.md            ← 프로젝트 (팀 공유 ✅)
src/payment/CLAUDE.md        ← 서브디렉토리 (더 높은 우선순위)
우선순위 낮음
```
- `@파일경로`: 다른 CLAUDE.md 파일 임포트

#### .claude/rules/
- YAML 파일 + glob 패턴 frontmatter
- 특정 파일 유형에만 조건부 적용
```yaml
---
paths:
  - "**/*.test.tsx"
  - "terraform/**/*"
---
```

#### 커맨드 vs Skills
| 위치 | 공유 범위 |
|------|---------|
| `.claude/commands/` | 팀 공유 (버전 관리) |
| `~/.claude/commands/` | 개인 전용 |

Skills frontmatter:
- `context: fork`: 격리 실행
- `allowed-tools`: 사용 가능 툴 목록
- `argument-hint`: 인자 없을 때 안내

#### Plan Mode 기준
- 사용: 45개+ 파일, 아키텍처 결정, 마이그레이션
- 불필요: 단일 파일 수정, 간단한 버그 수정

#### CI/CD
- `-p` / `--print`: 비대화형 모드 (필수!)
- `--output-format json`: 기계 처리 가능 출력
- `--json-schema`: 스키마 강제

---

## Domain 4: Prompt Engineering (20%)

### ✅ 반드시 알아야 할 주제

#### 명시적 기준 설계
- 모호한 지시 피하기: "보수적으로", "확신할 때만"
- 구체적 기준: "다음 조건을 모두 충족할 때만 X를 하라"
- False Positive: 일시 비활성화 → 기준 개발 → 재활성화

#### Few-Shot 패턴
- 2-4개 예시 (각각 다른 케이스)
- 모호한 케이스, 출력 형식, 다양한 구조 처리에 활용

#### 구조화된 출력
- `tool_use`: JSON 구문 오류 제거 (API 보장)
- 의미적 오류는 별도 검증 필요
- `nullable` 필드: 없는 정보 → null (환각 방지)

#### Batch API
- 50% 비용 절감
- 최대 24시간 처리
- SLA 없음 (지연 허용 시 사용)
- 적합: 야간 보고서, 대량 분류
- 부적합: pre-merge 체크, 실시간 응답

#### 리뷰 패턴
- 자기 리뷰 < 독립 인스턴스 리뷰
- 멀티패스: 파일별 분석 → 크로스파일 통합

---

## Domain 5: Context Management (15%)

### ✅ 반드시 알아야 할 주제

#### Lost-in-the-Middle
- 중간 정보 누락 위험
- 해결: 중요 정보를 처음과 끝에 배치
- 섹션 헤더로 구분

#### 요약 전략
- 수치(금액, 날짜, 번호) 반드시 보존
- "여러 주문" → "주문 #001 $45, #002 $120"
- 명시적 보존 지시 필요

#### 에스컬레이션 기준
즉시 에스컬레이션:
- 고객이 명시적으로 인간 요청
- 정책 공백/예외 상황
- 진전 불가한 상황

에스컬레이션 금지:
- 자신감 점수 낮음 ❌
- 고객이 화남 ❌
- 단순히 복잡해 보임 ❌

#### 에러 전파
구조화된 에러 컨텍스트:
- 에러 유형
- 시도된 작업
- 부분 결과
- 재시도 가능 여부
- 대안

#### 스크래치패드
- 긴 세션의 중간 상태 기록
- 컨텍스트 저하 방지

#### 상충 정보
- 임의 선택 금지
- 두 수치 모두 출처와 함께 보고

---

## 6개 시나리오 핵심 포인트

| 시나리오 | 핵심 개념 | 주요 함정 |
|---------|---------|---------|
| 1. 고객 지원 | 프로그래밍적 게이트 | 프롬프트로 순서 강제 불가 |
| 2. Claude Code | Plan Mode 판단 | 항상/절대 Plan Mode ❌ |
| 3. 멀티에이전트 연구 | 코디네이터 분해 | 좁은 분해 → 범위 누락 |
| 4. 개발자 생산성 | 툴 수 제한 | 더 많은 툴 = 더 나은 결과 ❌ |
| 5. CI/CD | -p 플래그 필수 | 대화형 모드로 파이프라인 ❌ |
| 6. 데이터 추출 | nullable + 재시도 | 없는 값 추측 ❌ |

---

> 🔗 다음 챕터: [시험 범위 외 주제](28_out_of_scope.md)
