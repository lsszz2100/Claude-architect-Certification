# Chapter 5: 멀티에이전트 시스템 설계

> 📅 2026년 04월 05일 기준  
> 🎯 **Domain 1 핵심 — Hub-and-Spoke 아키텍처**

---

## 5.1 왜 멀티에이전트가 필요한가?

### 단일 에이전트의 한계

```
단일 에이전트로 "전체 시장 조사 후 투자 보고서 작성"을 요청하면:
- 컨텍스트 창 초과 위험
- 웹 검색, 문서 분석, 수치 계산을 동시에 해야 함
- 각 단계 실패 시 전체 재시작 필요
- 병렬 처리 불가 → 매우 느림
```

### 멀티에이전트의 장점

```
코디네이터 에이전트
    ├── 웹 검색 에이전트 (병렬)
    ├── 문서 분석 에이전트 (병렬)
    └── 수치 분석 에이전트 (병렬)
            ↓ 결과 취합
    보고서 생성 에이전트
```

- **병렬 처리**: 독립적인 작업 동시 수행
- **전문화**: 각 에이전트가 특정 역할에 최적화
- **격리**: 한 에이전트 실패가 전체에 영향 없음
- **확장성**: 에이전트 추가로 기능 확장 용이

---

## 5.2 Hub-and-Spoke 아키텍처

> 🎯 **시험 최빈출 — 코디네이터-서브에이전트 패턴**

### 구조 개요

```
                  [코디네이터]
                      │
        ┌─────────────┼─────────────┐
        │             │             │
   [서브에이전트1] [서브에이전트2] [서브에이전트3]
   웹 검색         문서 분석      보고서 생성
```

**핵심 원칙:**
1. 코디네이터가 모든 서브에이전트 간 통신을 중재
2. 서브에이전트들은 서로 직접 통신하지 않음
3. 모든 정보는 코디네이터를 통해 흐름

### 핵심 특성 (시험 출제!)

> ⚠️ **서브에이전트는 코디네이터의 컨텍스트를 자동으로 상속받지 않습니다!**

```python
# ❌ 잘못된 이해: 서브에이전트가 자동으로 이전 내용 알 것이라 가정
subagent_task = "방금 분석한 내용을 바탕으로 요약해줘"
# 서브에이전트는 '방금 분석한 내용'이 무엇인지 모름!

# ✅ 올바른 방법: 컨텍스트를 명시적으로 전달
web_search_results = "구글 주가: $150, 매출 성장: 15%..."
document_analysis = "Q1 보고서: 영업이익 20% 증가..."

subagent_task = f"""
다음 데이터를 바탕으로 요약을 작성해주세요:

웹 검색 결과:
{web_search_results}

문서 분석 결과:
{document_analysis}

요약을 작성해주세요.
"""
```

---

## 5.3 컨텍스트 전달 전략

### 구조화된 컨텍스트 전달

```python
import json
from anthropic import Anthropic

client = Anthropic()

def run_research_pipeline(topic: str):
    """멀티에이전트 연구 파이프라인"""
    
    # Step 1: 웹 검색 에이전트 실행
    web_results = run_web_search_agent(topic)
    
    # Step 2: 문서 분석 에이전트 실행
    doc_analysis = run_document_analysis_agent(topic)
    
    # Step 3: 합성 에이전트에 모든 결과를 명시적으로 전달
    synthesis_prompt = f"""
    주제: {topic}
    
    ## 웹 검색 결과
    출처: {web_results['sources']}
    날짜: {web_results['dates']}
    내용: {web_results['content']}
    
    ## 문서 분석 결과  
    분석 문서: {doc_analysis['documents']}
    핵심 발견: {doc_analysis['findings']}
    
    위 정보를 종합하여 포괄적인 연구 보고서를 작성해주세요.
    각 주장에 대해 출처를 명시해주세요.
    """
    
    return run_synthesis_agent(synthesis_prompt)
```

### 메타데이터와 콘텐츠 분리

```python
# ✅ 좋은 구조화: 콘텐츠와 메타데이터 분리
structured_finding = {
    "claim": "2025년 AI 시장 규모는 1조 달러를 초과할 전망",
    "evidence": "IDC 보고서에 따르면 AI 소프트웨어 시장이 전년 대비 45% 성장",
    "source": {
        "url": "https://idc.com/report/2025",
        "document_name": "IDC AI Market Report 2025",
        "page": 12,
        "publication_date": "2025-01-15"
    },
    "confidence": 0.92
}
```

---

## 5.4 병렬 서브에이전트 실행

> 🎯 **시험 출제: Task 툴을 이용한 병렬 실행**

### Task 툴 설정

```python
# 코디네이터의 allowedTools에 반드시 "Task" 포함!
coordinator_tools = [
    {"name": "Task", "type": "custom"},  # ← 필수!
    {"name": "aggregate_results", "type": "custom"},
]
```

### 병렬 실행 구현

```python
# ✅ 올바른 병렬 실행: 한 번의 코디네이터 응답에서 여러 Task 호출
# 코디네이터 프롬프트에서:
coordinator_system = """
당신은 연구 코디네이터입니다. 
복잡한 연구 주제를 받으면 여러 서브에이전트를 동시에 실행하여 
빠르게 결과를 수집합니다.

사용 가능한 서브에이전트:
- web_search_agent: 최신 웹 정보 검색
- document_analyst: PDF/문서 분석  
- data_analyst: 수치 데이터 분석

중요: 독립적인 작업은 여러 Task를 동시에 호출하세요.
"""

# 코디네이터가 한 번의 응답으로 여러 Task를 호출:
# Task(web_search_agent, "AI 시장 현황 검색")
# Task(document_analyst, "첨부된 보고서 분석")  ← 동시 실행!
# Task(data_analyst, "매출 데이터 분석")
```

---

## 5.5 코디네이터 설계 원칙

### 좋은 코디네이터 프롬프트

```python
coordinator_prompt = """
당신은 "{topic}" 주제에 대한 연구를 지휘하는 코디네이터입니다.

## 역할
1. 주제를 분석하여 필요한 연구 영역 파악
2. 적절한 서브에이전트에 작업 위임
3. 결과를 평가하고 필요시 추가 조사 요청
4. 최종 종합 보고서 생성

## 품질 기준
- 모든 주장에는 출처가 있어야 함
- 상충되는 정보는 양측 모두 제시
- 데이터 날짜를 명시
- 확실하지 않은 정보는 불확실성 표시

## 중요 사항
- 단계적 지시가 아닌 목표와 품질 기준을 제시
- 서브에이전트의 적응성을 신뢰하되 결과를 검증
"""
```

### ⚠️ 좁은 태스크 분해의 위험

```python
# ❌ 잘못된 태스크 분해 (너무 좁음)
subtasks = [
    "AI in digital art creation",  # visual arts만!
    "AI in graphic design",         # 음악, 영화 누락
    "AI in photography"
]
# → 연구 범위가 편향됨 (시나리오 7번 문제 패턴)

# ✅ 올바른 태스크 분해 (포괄적)
subtasks = [
    "AI in visual arts (painting, photography, graphic design)",
    "AI in music composition and production",
    "AI in writing and literature",
    "AI in film and video production",
    "AI in performing arts"
]
```

---

## 📝 챕터 요약

- Hub-and-Spoke: 코디네이터가 모든 통신 중재
- 서브에이전트는 컨텍스트 자동 상속 ❌ → 명시적 전달 ✅
- 병렬 실행: 한 코디네이터 응답에서 여러 Task 동시 호출
- `allowedTools`에 `"Task"` 반드시 포함
- 태스크 분해가 너무 좁으면 중요한 영역 누락 가능

---

> 🔗 다음 챕터: [워크플로우 설계](06_workflow_design.md)
