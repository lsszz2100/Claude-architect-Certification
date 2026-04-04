# 부록 A: 핵심 용어 사전

> 📅 2026년 04월 05일 기준  
> 🔤 시험 전 최종 용어 확인


[← 마치며](32_conclusion.md) | [목차](../TOC.md) | [부록 B: 치트시트 →](A2_cheatsheet.md)

---

## A

### Agentic Loop (에이전틱 루프)
Claude가 목표 달성을 위해 반복적으로 생각하고 툴을 사용하는 사이클.
`stop_reason == "end_turn"` 시 종료.

### allowedTools
서브에이전트가 사용할 수 있는 툴 목록. `"Task"`를 포함해야 서브에이전트 스폰 가능.

### Anthropic
Claude를 개발하는 AI 안전 연구 회사. Constitutional AI 방법론 사용.

---

## B

### Batch API (Message Batches API)
비동기 대량 처리 API. 50% 비용 절감, 최대 24시간 처리, SLA 없음.

### business (errorCategory)
정책 위반, 한도 초과 등 비즈니스 규칙으로 인한 오류. `isRetryable: false`.

---

## C

### CLAUDE.md
Claude Code에서 프로젝트 가이드라인을 정의하는 파일.
계층: 사용자 → 프로젝트 → 서브디렉토리.

### Constitutional AI (CAI)
Anthropic의 AI 안전 훈련 방법론. 원칙 기반 학습.

### context: fork
Skills frontmatter 옵션. 현재 세션과 격리된 서브에이전트에서 실행.

### Coordinator (코디네이터)
멀티에이전트 시스템에서 전체를 조율하는 에이전트. Hub 역할.

---

## D

### Direct Execution
Claude Code에서 계획 없이 바로 코드를 수정하는 모드. 단순한 변경에 적합.

### Dynamic Decomposition (동적 분해)
LLM이 판단하여 태스크를 유연하게 분해하는 방법. 미지의 범위에 적합.

---

## E

### end_turn
에이전틱 루프의 정상 종료 신호. `stop_reason == "end_turn"` 시 루프 종료.

### errorCategory
에러 분류: transient, validation, business, permission.

---

## F

### False Positive (FP)
실제로 문제가 없는데 문제라고 잘못 감지하는 경우.
해결: 해당 카테고리 일시 비활성화 → 기준 개발 → 재활성화.

### Few-Shot
2-4개의 예시를 제공하여 LLM이 원하는 패턴을 학습하게 하는 기법.

### fork_session
공통 기준점에서 독립적인 여러 탐색 경로를 시작하는 세션 관리 기능.

---

## G

### Glob
파일 경로 패턴으로 파일을 찾는 내장 툴. `**/*.py` 등의 패턴 사용.

### Grep
파일 내용에서 패턴을 검색하는 내장 툴. 정규식 지원.

---

## H

### Hallucination (환각)
LLM이 없는 사실을 생성하는 현상. nullable 필드로 방지.

### Hub-and-Spoke
코디네이터(Hub)가 여러 서브에이전트(Spokes)를 관리하는 멀티에이전트 아키텍처.

### Hooks
툴 실행 전(PreToolUse) 또는 후(PostToolUse)에 코드를 삽입하는 메커니즘.

---

## I

### isRetryable
에러 응답의 필드. true: 재시도 가능(transient), false: 재시도 불가.

---

## L

### Lost-in-the-Middle
LLM이 긴 컨텍스트에서 중간 정보를 놓치는 현상.
대응: 중요 정보를 처음과 끝에 배치.

---

## M

### MCP (Model Context Protocol)
Claude와 외부 시스템을 연결하는 표준 프로토콜.

### .mcp.json
프로젝트 범위 MCP 서버 설정 파일. 팀 공유, 버전 관리.

### Multi-Agent (멀티에이전트)
여러 Claude 인스턴스가 협력하여 복잡한 작업을 수행하는 시스템.

---

## N

### nullable
JSON 스키마에서 null 값을 허용하는 필드 설정. `{"type": ["string", "null"]}`.
환각 방지에 필수.

---

## P

### permission (errorCategory)
인증 실패, 접근 거부 등 권한 관련 오류. `isRetryable: false`.

### Plan Mode
대규모 변경 전에 계획을 먼저 수립하는 Claude Code 모드.

### PostToolUse
툴 실행 후 결과를 변환/정규화하는 Hook.

### PreToolUse
툴 실행 전 차단/정책 강제를 수행하는 Hook.

### Prompt Chaining (프롬프트 체이닝)
순차적으로 고정된 워크플로우. 각 단계가 이전 결과에 의존.

---

## S

### Skills
YAML frontmatter가 있는 고급 슬래시 커맨드.
옵션: context, allowed-tools, argument-hint.

### stop_reason
Claude 응답이 종료된 이유. end_turn, tool_use, max_tokens, stop_sequence.

### Subagent (서브에이전트)
코디네이터가 스폰한 특정 작업 담당 에이전트. 코디네이터 컨텍스트 자동 상속 없음.

---

## T

### Task (툴)
서브에이전트를 스폰하는 툴. allowedTools에 반드시 포함 필요.

### token_use
구조화된 출력을 보장하는 API 기능. JSON 구문 오류 제거.

### tool_choice
API가 툴을 선택하는 방식. auto/any/특정 툴 강제.

### transient (errorCategory)
타임아웃, 서버 오류 등 일시적 오류. `isRetryable: true`.

---

## V

### validation (errorCategory)
잘못된 입력 형식, 필수 필드 누락 등 입력 오류. `isRetryable: false`.

---

## W

### WikiDocs
한국형 위키 문서 플랫폼. GitHub 통합으로 마크다운 파일을 책으로 출판.

---

> 🔗 관련: [부록 B: 핵심 치트시트](A2_cheatsheet.md)
