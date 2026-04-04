# Chapter 06: 워크플로우 설계

> 📅 2026년 04월 05일 기준  
> 🎯 프로그래밍적 강제 vs 프롬프트 지시 — 시험 핵심


[← Chapter 05](05_multi_agent.md) | [목차](../TOC.md) | [Chapter 07: 툴 설계 →](07_tool_design.md)

---

## 06.1 프로그래밍적 강제 vs 프롬프트 지시

> 🎯 시험 최빈출 개념 — 반드시 이해

### 핵심 원칙

```
프롬프트 지시 = 확률적 준수 (때로는 무시됨)
프로그래밍적 강제 = 결정론적 보장 (항상 적용됨)
```

### 언제 무엇을 사용하나?

| 상황 | 권장 방법 | 이유 |
|------|-----------|------|
| 금융 거래 전 본인 확인 | 프로그래밍적 강제 | 실패 시 금전 피해 |
| 개인정보 처리 전 동의 | 프로그래밍적 강제 | 법적 요구사항 |
| 응답 스타일 가이드 | 프롬프트 지시 | 약간의 변형 허용 가능 |
| 언어 선택 | 프롬프트 지시 | 유연성 필요 |

### 실제 구현 비교

```python
# ❌ 잘못된 방법: 프롬프트만으로 순서 강제
system_prompt = """
중요: 반드시 get_customer를 먼저 호출한 후에만 
process_refund를 호출하세요.
"""
# → 12%의 경우에 무시됨 (실제 시험 시나리오 수치!)

# ✅ 올바른 방법: 프로그래밍적 게이트
class RefundWorkflow:
    def __init__(self):
        self.customer_verified = False
        self.customer_id = None
    
    def execute_tool(self, tool_name: str, tool_input: dict):
        """툴 실행 전 전제 조건 확인"""
        
        # 게이트: customer 검증 전에는 refund 불가
        if tool_name in ["process_refund", "lookup_order"]:
            if not self.customer_verified:
                return {
                    "error": "고객 확인이 먼저 필요합니다.",
                    "required_action": "get_customer 먼저 호출하세요",
                    "isError": True
                }
        
        # 실제 툴 실행
        if tool_name == "get_customer":
            result = self._get_customer(tool_input)
            if result.get("verified"):
                self.customer_verified = True
                self.customer_id = result["customer_id"]
            return result
        
        elif tool_name == "process_refund":
            return self._process_refund(tool_input, self.customer_id)
        
        # ... 기타 툴
```

---

## 06.2 Agent SDK Hooks

> 🎯 시험 출제: PostToolUse 훅 패턴

### 훅이란?

훅(Hook)은 툴 호출 전후에 자동으로 실행되는 인터셉터입니다.

```
툴 호출 요청
    ↓
[PreToolUse 훅] ← 툴 호출 차단 가능
    ↓
툴 실행
    ↓
[PostToolUse 훅] ← 결과 변환 가능
    ↓
Claude에 결과 전달
```

### PostToolUse 훅: 데이터 정규화

```python
def post_tool_use_hook(tool_name: str, tool_result: dict) -> dict:
    """
    PostToolUse 훅: 이기종 MCP 툴의 데이터 형식을 정규화
    """
    
    if tool_name == "get_customer":
        # Unix 타임스탬프를 ISO 8601로 변환
        if "created_at" in tool_result:
            timestamp = tool_result["created_at"]
            tool_result["created_at"] = datetime.fromtimestamp(
                timestamp
            ).isoformat()
        
        # 숫자 상태 코드를 문자열로 변환
        status_map = {1: "active", 2: "suspended", 3: "closed"}
        if "status" in tool_result:
            tool_result["status_text"] = status_map.get(
                tool_result["status"], "unknown"
            )
    
    return tool_result


def pre_tool_use_hook(tool_name: str, tool_input: dict):
    """
    PreToolUse 훅: 비즈니스 규칙 강제
    """
    
    if tool_name == "process_refund":
        amount = tool_input.get("amount", 0)
        
        # $500 초과 환불은 인간 승인 필요
        if amount > 500:
            # 툴 실행 차단 및 에스컬레이션
            raise PolicyViolationError(
                f"환불 금액 ${amount}이 자동 처리 한도($500)를 초과합니다.",
                action="escalate_to_human",
                amount=amount
            )
```

### 훅 vs 프롬프트 지시 선택 기준

```
비즈니스 규칙이 반드시 지켜져야 하는가?
├── YES → 훅 사용 (결정론적 보장)
└── NO  → 프롬프트 지시로 충분
```

---

## 06.3 태스크 분해 전략

### 두 가지 분해 패턴

1. 프롬프트 체이닝 (Prompt Chaining)
- 예측 가능한 다단계 작업에 적합
- 각 단계가 독립적이고 순서가 명확할 때

```python
# 코드 리뷰 파이프라인 (프롬프트 체이닝)
def review_pipeline(pr_files: list):
    results = []
    
    # Step 1: 각 파일 개별 분석 (로컬 이슈)
    for file in pr_files:
        local_issues = analyze_file_locally(file)
        results.append(local_issues)
    
    # Step 2: 파일 간 통합 분석 (크로스파일 이슈)
    integration_issues = analyze_cross_file_issues(results)
    
    # Step 3: 최종 보고서 생성
    return generate_review_report(results, integration_issues)
```

2. 동적 적응 분해 (Dynamic Decomposition)
- 오픈엔디드 조사 작업에 적합
- 이전 단계 결과에 따라 다음 단계 결정

```python
# 레거시 코드베이스 테스트 추가 (동적 분해)
def add_tests_to_legacy_codebase():
    # Step 1: 코드베이스 구조 파악
    structure = map_codebase_structure()
    
    # Step 2: 고임팩트 영역 식별 (발견에 기반)
    high_impact_areas = identify_high_impact_areas(structure)
    
    # Step 3: 발견된 의존성 기반 우선순위 계획
    test_plan = create_prioritized_plan(high_impact_areas)
    
    # Step 4: 계획에 따라 순차 실행
    return execute_test_plan(test_plan)
```

---

## 06.4 세션 관리와 fork_session

### 세션 재개 (--resume)

```bash
# 이름 있는 세션 시작
claude --session-name "payment-bug-investigation"

# 이전 세션 재개
claude --resume "payment-bug-investigation"
```

### 세션 재개 시 주의사항

```python
# ❌ 잘못된 방법: 파일이 변경된 후 세션 재개 시 알리지 않음
# (에이전트가 오래된 파일 내용을 기억하고 있을 수 있음)

# ✅ 올바른 방법: 변경된 파일을 명시적으로 알림
resume_context = """
세션을 재개합니다. 마지막 세션 이후 변경 사항:
- payment/processor.py: 환불 로직 수정됨 (재검토 필요)
- tests/test_payment.py: 새 테스트 케이스 추가됨
이전 분석 내용을 바탕으로 변경 사항을 반영해주세요.
"""
```

### fork_session 활용

```python
# 공유 분석 기준점에서 독립 브랜치 생성
# 예: 두 가지 리팩토링 접근법 비교

base_analysis = analyze_codebase()  # 공통 분석

# 접근법 A를 독립적으로 탐색
fork_session_a = fork_session(base_analysis)
approach_a = explore_refactoring_approach_a(fork_session_a)

# 접근법 B를 독립적으로 탐색 (base_analysis에서 시작)
fork_session_b = fork_session(base_analysis)
approach_b = explore_refactoring_approach_b(fork_session_b)

# 두 결과 비교
compare_approaches(approach_a, approach_b)
```

---

## 📝 챕터 요약

- 프로그래밍적 강제 > 프롬프트 지시: 금융, 법규 등 critical 비즈니스 규칙
- PostToolUse 훅: 이기종 데이터 형식 정규화 (Unix ts → ISO 8601)
- PreToolUse 훅: 비즈니스 규칙 위반 차단 ($500 초과 환불 차단 등)
- 프롬프트 체이닝: 예측 가능한 순서 작업
- 동적 분해: 발견에 기반한 적응적 작업
- fork_session: 공통 기준점에서 독립 탐색

---

> 🔗 다음 챕터: [효과적인 툴 설계](07_tool_design.md)
