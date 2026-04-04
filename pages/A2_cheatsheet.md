# 부록 B: 핵심 치트시트

> 📅 2026년 04월 05일 기준  
> 🎯 시험 직전 최종 확인용

---

## Domain 1: Agentic Architecture (27%)

### 에이전틱 루프 제어
```python
# 올바른 루프 종료
if response.stop_reason == "end_turn":    # 종료
    break
elif response.stop_reason == "tool_use":  # 툴 실행 후 계속
    ...

# ❌ 절대 하면 안 되는 것
if "DONE" in response.text:   # 텍스트로 판단 금지
if i >= 10:                    # 고정 횟수로만 종료 금지
```

### 멀티에이전트 핵심 규칙
- 서브에이전트 = 코디네이터 컨텍스트 자동 상속 ❌
- `allowedTools`에 `"Task"` 반드시 포함
- 병렬 실행 = 한 응답에서 여러 Task 동시 호출

### 프로그래밍적 강제 vs 프롬프트
```
Critical 비즈니스 로직 → 프로그래밍적 강제 (결정론적)
스타일/선호도 → 프롬프트 지시 (확률적)
```

### Hooks
- `PostToolUse`: 툴 결과 변환/정규화
- `PreToolUse(before)`: 툴 호출 차단/정책 강제

### 세션 관리
- `--resume <session-name>`: 이름 있는 세션 재개
- `fork_session`: 공통 기준점에서 독립 탐색

---

## 🔧 Domain 2: Tool Design & MCP (18%)

### 툴 설명 원칙
- 툴 설명 = LLM 툴 선택의 1차적 메커니즘
- 포함할 것: 입력 형식, 예제 쿼리, 엣지 케이스, 경계
- 비슷한 툴은 명확히 구분

### 에러 분류
| errorCategory | isRetryable | 예시 |
|--------------|-------------|------|
| transient | ✅ | 타임아웃, 서버 장애 |
| validation | ❌ | 잘못된 입력 형식 |
| business | ❌ | 정책 위반, 한도 초과 |
| permission | ❌ | 권한 없음 |

### tool_choice
- `"auto"`: 자율 선택 (기본)
- `"any"`: 반드시 툴 호출
- `{"type": "tool", "name": "..."}`: 특정 툴 강제

### MCP 범위
- `.mcp.json` (프로젝트) → 팀 공유, 버전 관리
- `~/.claude.json` (사용자) → 개인 전용

### 내장 툴 선택
- Grep: 파일 내용 검색 (코드에서 패턴 찾기)
- Glob: 파일 경로 패턴 (*.test.tsx 등)
- Read: 전체 파일 읽기
- Edit: 유니크 텍스트 교체 (실패 시 Read+Write)
- Write: 전체 파일 재작성

---

## 💻 Domain 3: Claude Code (20%)

### CLAUDE.md 계층
```
~/.claude/CLAUDE.md     ← 사용자 (팀 공유 ❌)
.claude/CLAUDE.md       ← 프로젝트 (팀 공유 ✅)
src/payment/CLAUDE.md   ← 디렉토리 전용
```

### .claude/rules/ 규칙
```yaml
---
paths:
  - "**/*.test.tsx"  # glob 패턴
  - "terraform/**/*"
---
```

### 커맨드 vs Skills
- `.claude/commands/`: 팀 공유 슬래시 커맨드
- `~/.claude/commands/`: 개인 커맨드
- `context: fork`: 격리된 서브에이전트에서 실행
- `allowed-tools`: 사용 가능한 툴 제한

### Plan Mode 사용 기준
```
Plan Mode:
  ✅ 45개+ 파일 변경
  ✅ 아키텍처 결정
  ✅ 마이크로서비스 분리
  ✅ 라이브러리 마이그레이션

Direct Execution:
  ✅ 단일 파일 버그 수정
  ✅ 명확한 스택 트레이스
  ✅ 간단한 함수 추가
```

### CI/CD
- `-p` (`--print`): 비대화형 모드 (파이프라인 필수!)
- `--output-format json`: 기계 처리 가능 출력
- `--json-schema`: 스키마 강제

---

## ✍️ Domain 4: Prompt Engineering (20%)

### False Positive 해결
1. 명시적 기준 (어떤 것을 보고하고 안 하는지)
2. FP 높은 카테고리 일시 비활성화
3. 심각도별 구체적 코드 예시

### Few-Shot 최적 사용
- 모호한 케이스 처리
- 출력 형식 일관성
- 다양한 문서 구조 처리
- 2-4개로 충분 (각각 다른 케이스)

### tool_use 구조화 출력
- JSON 구문 오류 제거 ✅
- 의미적 오류 (합계 불일치) → 별도 검증 필요
- `nullable`: 없을 수 있는 정보 → null (환각 방지)

### Batch API
| 항목 | 값 |
|------|-----|
| 비용 절감 | 50% |
| 처리 시간 | 최대 24시간 |
| SLA | 없음 |
| 적합 | 야간 보고서, 주간 분석 |
| 부적합 | pre-merge 체크, 실시간 응답 |

### 멀티패스 리뷰
- 자기 리뷰 < 독립 인스턴스 리뷰
- 파일별 로컬 분석 → 크로스파일 통합 분석

---

## 🧠 Domain 5: Context Management (15%)

### Lost-in-the-Middle
- 중간 정보 누락 위험
- 중요 정보 → 처음/끝에 배치
- 섹션 헤더로 구분

### 요약 시 보존 필수
- 금액, 날짜, 주문번호, 고객 ID
- "여러 주문" ❌ → "주문 #001 $45, 주문 #002 $120" ✅

### 에스컬레이션 기준
```
즉시 에스컬레이션:
  ✅ 고객이 명시적으로 사람 요청
  ✅ 정책 공백/예외 상황

에스컬레이션 불필요:
  ❌ 단순히 복잡해 보임
  ❌ 자신감 낮음
  ❌ 고객이 화남 (감정 기반)
```

### 에러 전파
```python
# 구조화된 에러 컨텍스트
{
  "errorType": "timeout",
  "attemptedQuery": query,
  "partialResults": [...],
  "isRetryable": True,
  "suggestedAlternatives": [...]
}
```

---

## 시험 당일 기억할 것

### 문제 풀기 전 체크
1. 시나리오의 도메인 파악
2. 핵심 제약 조건 확인
3. 각 선택지의 근본 원인 분석

### 정답 패턴

| 상황 | 정답 경향 |
|------|----------|
| Critical 비즈니스 로직 | 프로그래밍적 강제 |
| 툴 선택 오류 | 툴 설명 개선 (first step) |
| 에스컬레이션 조정 | 명시적 기준 + few-shot |
| 팀 커맨드 위치 | `.claude/commands/` |
| CI/CD 비대화형 | `-p` 플래그 |
| 비차단 대량 작업 | Batch API |
| 구조화 출력 | tool_use |
| 자기 리뷰 품질 | 독립 인스턴스 사용 |

---

*이 치트시트는 시험 직전 최종 복습용입니다. 개념의 깊은 이해가 있어야 올바르게 적용할 수 있습니다.*
