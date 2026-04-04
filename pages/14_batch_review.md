# Chapter 14: 배치 처리와 리뷰 아키텍처

> 📅 2026년 04월 05일 기준  
> 🎯 **Domain 4 — Batch API 50% 비용 절감, 차단/비차단 구분**

---

## 14.1 Message Batches API

> 🎯 **시험 핵심: Batch API는 비차단 워크플로우에만 적합**

### Batch API 특성

| 특성 | 값 |
|------|-----|
| 비용 절감 | **50%** |
| 처리 시간 | 최대 **24시간** |
| 지연 SLA | **없음** (보장 안 됨) |
| 멀티턴 툴 호출 | **지원 안 됨** |

### 언제 Batch API를 쓰나?

```
비차단 워크플로우 (Batch API 적합):
✅ 야간 기술 부채 보고서
✅ 주간 코드 품질 분석
✅ 대량 문서 분류 (하룻밤)
✅ 레거시 코드 분석

차단 워크플로우 (실시간 API 필수):
❌ 머지 전 코드 리뷰 (개발자가 대기 중)
❌ 고객 응답 (실시간 필요)
❌ CI/CD 게이트 (파이프라인 차단)
```

### Batch API 구현

```python
import anthropic
import json
import time

client = anthropic.Anthropic()

def batch_process_invoices(invoices: list[dict]) -> dict:
    """대량 송장 처리 (비용 최적화)"""
    
    # 배치 요청 생성
    requests = []
    for invoice in invoices:
        requests.append({
            "custom_id": invoice["id"],  # 나중에 결과 매핑용
            "params": {
                "model": "claude-sonnet-4-6",
                "max_tokens": 1024,
                "tools": [extraction_tool],
                "tool_choice": {"type": "tool", "name": "extract_invoice_data"},
                "messages": [{
                    "role": "user",
                    "content": f"다음 송장을 처리해주세요:\n{invoice['text']}"
                }]
            }
        })
    
    # 배치 제출
    batch = client.beta.messages.batches.create(requests=requests)
    print(f"배치 제출 완료: {batch.id}")
    
    # 완료 대기 (폴링)
    while True:
        batch = client.beta.messages.batches.retrieve(batch.id)
        if batch.processing_status == "ended":
            break
        time.sleep(60)  # 1분마다 확인
    
    # 결과 처리
    results = {}
    failed_ids = []
    
    for result in client.beta.messages.batches.results(batch.id):
        if result.result.type == "succeeded":
            results[result.custom_id] = result.result.message
        else:
            failed_ids.append(result.custom_id)
            print(f"실패: {result.custom_id} - {result.result.error}")
    
    # 실패한 항목 재처리 (예: 컨텍스트 초과 → 청킹)
    if failed_ids:
        print(f"{len(failed_ids)}개 실패. 재처리 중...")
        for fail_id in failed_ids:
            invoice = next(i for i in invoices if i["id"] == fail_id)
            # 청킹 처리
            results[fail_id] = process_chunked(invoice)
    
    return results


def calculate_batch_schedule(sla_hours: int, batch_max_hours: int = 24) -> int:
    """SLA를 맞추기 위한 배치 제출 주기 계산"""
    # 예: SLA 30시간, 배치 최대 24시간
    # → 6시간마다 배치 제출해야 안전
    submission_interval = sla_hours - batch_max_hours
    return max(1, submission_interval)  # 최소 1시간

# 30시간 SLA의 경우
interval = calculate_batch_schedule(sla_hours=30)
print(f"배치 제출 주기: {interval}시간")  # → 6시간
```

---

## 14.2 멀티패스 리뷰 설계

> 🎯 **시험 출제: 자기 리뷰의 한계, 독립 인스턴스가 더 효과적**

### 자기 리뷰의 한계

```python
# ❌ 잘못된 방법: 생성한 세션에서 바로 리뷰
generated_code = generate_code(requirements)  # 세션 A
review = review_code(generated_code)  # 같은 세션 A → 편향!

# 문제: Claude가 생성 당시의 추론 컨텍스트를 기억
# → 자신의 결정에 의문을 갖지 않음
# → 미묘한 버그를 놓칠 가능성 높음

# ✅ 올바른 방법: 독립 인스턴스로 리뷰
generated_code = generate_code(requirements)  # 세션 A
# 새 Claude 인스턴스 (이전 맥락 없음)
review = independent_review(generated_code)  # 세션 B → 더 객관적!
```

### 14파일 PR 리뷰 멀티패스 구조

```python
# 시나리오: 14개 파일의 PR이 있음
# 단일 패스로는 주의력 분산 → 오류 놓침

def review_large_pr(pr_files: list[str]) -> dict:
    """멀티패스 코드 리뷰"""
    
    all_issues = []
    
    # Pass 1: 파일별 로컬 이슈 분석
    file_issues = {}
    for file_path in pr_files:
        file_content = read_file(file_path)
        issues = analyze_single_file(file_path, file_content)
        file_issues[file_path] = issues
        all_issues.extend(issues)
    
    # Pass 2: 파일 간 통합 이슈 분석 (별도 Claude 인스턴스)
    integration_issues = analyze_cross_file_issues(
        files=pr_files,
        individual_results=file_issues
        # 이 단계는 이전 파일별 결과를 입력으로 받되,
        # 생성 세션과는 독립적임
    )
    all_issues.extend(integration_issues)
    
    return {
        "file_issues": file_issues,
        "integration_issues": integration_issues,
        "summary": generate_summary(all_issues)
    }


def analyze_cross_file_issues(files: list, individual_results: dict) -> list:
    """크로스파일 데이터 흐름 분석"""
    
    # 개별 파일 결과를 컨텍스트로 제공
    context = "\n\n".join([
        f"=== {file} ===\n{result}"
        for file, result in individual_results.items()
    ])
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=4096,
        messages=[{
            "role": "user",
            "content": f"""
다음은 각 파일의 개별 분석 결과입니다:

{context}

이제 파일 간 데이터 흐름과 통합 이슈를 분석해주세요:
1. 여러 파일에 걸친 데이터 불일치
2. 파일 간 의존성 문제
3. 한 파일의 변경이 다른 파일에 미치는 영향
"""
        }]
    )
    
    return parse_integration_issues(response.content[0].text)
```

### 신뢰도 기반 라우팅

```python
def review_with_confidence(code: str) -> dict:
    """신뢰도 포함 리뷰 → 인간 검토 우선순위화"""
    
    response = client.messages.create(
        ...,
        messages=[{
            "role": "user",
            "content": f"""
코드를 리뷰하고 각 이슈에 신뢰도(0.0-1.0)를 포함하세요.
신뢰도 기준:
- 1.0: 확실한 버그 (재현 가능한 시나리오 있음)
- 0.7-0.9: 높은 확률의 이슈
- 0.4-0.6: 의심스러운 패턴
- 0.3 미만: 가능성 낮음 (보고하지 않는 것 권장)

```python
{code}
```
"""
        }]
    )
    
    issues = parse_issues_with_confidence(response)
    
    # 신뢰도 기반 라우팅
    high_confidence = [i for i in issues if i["confidence"] >= 0.8]
    needs_human_review = [i for i in issues if 0.4 <= i["confidence"] < 0.8]
    
    return {
        "auto_flagged": high_confidence,      # 자동 처리
        "human_review": needs_human_review,    # 인간 검토 필요
        "ignored": [i for i in issues if i["confidence"] < 0.4]
    }
```

---

## 📝 챕터 요약

- Batch API: 50% 비용 절감, 최대 24시간, SLA 없음 → 비차단 작업만
- 차단 워크플로우(pre-merge 체크)는 실시간 API 필수
- 자기 리뷰 = 편향 → 독립 Claude 인스턴스로 리뷰가 더 효과적
- 14파일 PR → 파일별 로컬 분석 + 크로스파일 통합 분석 (2패스)
- 신뢰도 점수로 인간 리뷰 우선순위 설정

---

> 🔗 다음 챕터: [컨텍스트 관리 전략](15_context_management.md)
