# Chapter 16: 에스컬레이션과 신뢰성

> 📅 2026년 04월 05일 기준  
> 🎯 Domain 5 — 언제 에스컬레이션하는가?


[← Chapter 15](15_context_management.md) | [목차](../TOC.md) | [Chapter 17: 시나리오 1 →](17_scenario1_customer_support.md)

---

## 16.1 에스컬레이션 패턴 설계

> 🎯 시험 핵심: 명시적 기준 + few-shot이 가장 효과적

### 올바른 에스컬레이션 트리거

```
에스컬레이션해야 하는 경우:
✅ 고객이 명시적으로 사람과 통화하고 싶다고 요청
✅ 정책에 명시되지 않은 예외 상황 (정책 공백)
✅ 의미 있는 진전이 불가능한 상태
✅ 법적/규정 관련 문제

에스컬레이션하면 안 되는 경우:
❌ 단순히 복잡해 보이는 케이스
❌ 자신감(confidence score)이 낮을 때
❌ 고객이 불만족스러워 보일 때 (감정 기반 판단)
```

### 시스템 프롬프트에 명시적 기준 포함

```python
support_agent_prompt = """
당신은 ARIA 고객 지원 에이전트입니다.

## 에스컬레이션 기준 (필수 준수)

### 즉시 에스컬레이션 (이유 없이 바로)
- 고객이 "사람과 통화하고 싶다" 또는 이에 상응하는 표현을 명시할 때
- 법적 조치, 언론 제보 등을 언급할 때

### 에스컬레이션 필요 (정책 판단)
- 자사 웹사이트에 없는 타사 최저가 매칭 요청
  (우리 정책은 자사 이전 가격 매칭만 다룸)
- 천재지변/코로나 등 특수 상황에 의한 배송 지연 보상
  (정책이 명시적으로 다루지 않음)

### 에스컬레이션 불필요 (직접 해결)
- 표준 손상 제품 교환 (사진 증거 있음)
- 일반 배송 지연 환불 ($50 이하)
- 계정 정보 수정

## 중요: 에스컬레이션 순서
1. 고객 ID, 주문 번호, 이슈 요약, 이미 시도한 조치를 포함한
   구조화된 핸드오프 요약을 먼저 작성하세요
2. 상담원이 대화 기록에 접근하지 못할 수 있으므로
   모든 관련 정보를 요약에 포함하세요

## Few-Shot 예시

### 예시 1: 즉시 에스컬레이션
고객: "이 문제 정말 지겨워요. 제발 사람 좀 연결해주세요."
행동: 즉시 에스컬레이션
이유: 고객이 명시적으로 사람 연결을 요청함

### 예시 2: 독자적 해결 후 에스컬레이션 여부 확인
고객: "배송이 너무 늦었어요! 화가 나요!"
행동: 사과 후 주문 확인, 해결책 제안 → 고객이 거부하면 에스컬레이션
이유: 감정 표현이 있지만 사람 연결 명시 요청은 없음
"""
```

### 다중 고객 매칭 처리

```python
def handle_customer_lookup(name: str, results: list) -> dict:
    """여러 고객이 매칭될 때 처리"""
    
    if len(results) == 0:
        return {"action": "inform_not_found"}
    
    elif len(results) == 1:
        return {"action": "proceed", "customer": results[0]}
    
    else:
        # ❌ 잘못된 방법: 휴리스틱으로 선택
        # return {"action": "proceed", "customer": results[0]}  # 첫 번째 선택!
        
        # ✅ 올바른 방법: 추가 식별자 요청
        return {
            "action": "request_more_info",
            "message": f"'{name}'으로 {len(results)}명의 고객을 찾았습니다. 다음 중 하나를 알려주세요: 이메일 주소, 주문 번호, 전화번호",
            "candidates_count": len(results)
        }
```

---

## 16.2 에러 전파 전략

### 구조화된 에러 컨텍스트

```python
def web_search_agent(query: str):
    """웹 검색 서브에이전트"""
    
    try:
        results = search_web(query)
        return {
            "success": True,
            "results": results,
            "query": query,
            "sources": [r["url"] for r in results]
        }
    
    except TimeoutError:
        # ✅ 구조화된 에러 컨텍스트 반환
        return {
            "success": False,
            "errorType": "timeout",
            "attemptedQuery": query,
            "partialResults": [],  # 있다면 부분 결과 포함
            "isRetryable": True,
            "suggestedAlternatives": [
                f"{query} site:wikipedia.org",
                f"{query} filetype:pdf"
            ],
            "message": "웹 검색 타임아웃. 검색어 수정 또는 재시도 고려"
        }
```

---

## 16.3 대규모 코드베이스 탐색

### 스크래치패드 패턴

```python
SCRATCHPAD_FILE = ".claude/session_notes.md"

def explore_with_scratchpad(codebase_path: str):
    """스크래치패드로 긴 탐색 세션 관리"""
    
    # 주요 발견사항을 파일에 기록
    findings = {
        "entry_points": [],
        "key_classes": [],
        "dependencies": [],
        "potential_issues": []
    }
    
    # 탐색 중 발견 시마다 스크래치패드 업데이트
    def update_scratchpad(category: str, finding: str):
        findings[category].append(finding)
        with open(SCRATCHPAD_FILE, "w") as f:
            yaml.dump(findings, f)
    
    # 이후 질문 시 스크래치패드 참조
    with open(SCRATCHPAD_FILE) as f:
        previous_findings = yaml.safe_load(f)
    
    # 컨텍스트 저하 방지
    return previous_findings
```

### /compact 활용

```bash
# 긴 탐색 세션에서 컨텍스트가 차오를 때
/compact
# → Claude가 현재까지의 대화를 압축하여 컨텍스트 확보
```

---

## 16.4 정보 출처 보존

### 클레임-소스 매핑

```python
def require_source_attribution(agent_output: dict) -> bool:
    """모든 주장에 출처가 있는지 검증"""
    
    for finding in agent_output.get("findings", []):
        if not finding.get("source"):
            return False  # 출처 없는 주장 거부
        
        # 출처 메타데이터 검증
        source = finding["source"]
        required_fields = ["url_or_doc_name", "date", "excerpt"]
        for field in required_fields:
            if not source.get(field):
                return False
    
    return True


# 서브에이전트에 요구할 출력 구조
structured_finding_schema = {
    "claim": "주장 내용",
    "evidence": "증거 텍스트",
    "source": {
        "url": "https://...",
        "document_name": "문서명",
        "page": 12,
        "publication_date": "2025-01-15",
        "excerpt": "원문 발췌"
    },
    "confidence": 0.92,
    "temporal_note": "2025년 1월 기준 데이터"
}
```

### 상충 정보 처리

```python
# ❌ 잘못된 방법: 하나를 임의로 선택
conflicting_stats = [
    {"value": "45% 성장", "source": "IDC 2025"},
    {"value": "38% 성장", "source": "Gartner 2025"}
]
final_stat = conflicting_stats[0]["value"]  # 임의 선택!

# ✅ 올바른 방법: 두 수치 모두 보존하고 주석 달기
report_section = """
## AI 시장 성장률 (2025)

> ⚠️ 출처에 따라 다른 수치가 보고됩니다:
> - IDC (2025.01): **45% 성장** 전망
> - Gartner (2025.01): **38% 성장** 전망
> 
> 방법론 차이에 의한 것으로 추정되며, 
> 결정 시 두 수치 모두 고려하시기 바랍니다.
"""
```

---

## 📝 챕터 요약

- 에스컬레이션 기준: 명시적 요청, 정책 공백, 진전 불가 → 즉시 에스컬레이션
- 감정, 자신감 점수, 복잡도만으로는 에스컬레이션하지 말 것
- 다중 매칭: 임의 선택 대신 추가 식별자 요청
- 에러 전파: 실패 유형 + 시도한 내용 + 부분 결과 + 대안 포함
- 스크래치패드: 긴 세션에서 컨텍스트 저하 방지
- 상충 정보: 임의 선택 대신 두 수치 모두 출처와 함께 보존

---

> 🔗 다음 챕터: [시나리오 1 — 고객 지원 에이전트](17_scenario1_customer_support.md)
