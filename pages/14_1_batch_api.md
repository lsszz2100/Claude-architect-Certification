# 14.1 Message Batches API

> 📅 2026년 04월 05일 기준  
> ⭐ **시험 핵심 — 특성 암기 필수**

---

## Batch API 핵심 특성

```
비용: 표준 대비 50% 절감
처리 시간: 최대 24시간
SLA: 없음 (지연 허용)
최대 배치 크기: 요청당 최대 10,000개
```

---

## 언제 Batch API를 사용하는가?

```
✅ 적합:
- 야간 보고서 생성
- 주간 코드베이스 분석
- 대량 문서 분류
- 비실시간 데이터 처리

❌ 부적합:
- pre-merge 코드 체크 (즉각 응답 필요)
- 실시간 고객 지원
- 사용자가 대기하는 작업
```

---

## 구현

```python
import anthropic

client = anthropic.Anthropic()

# 배치 생성
batch = client.messages.batches.create(
    requests=[
        {
            "custom_id": f"doc-{i}",
            "params": {
                "model": "claude-sonnet-4-6",
                "max_tokens": 1024,
                "messages": [{
                    "role": "user",
                    "content": f"문서 분류: {doc}"
                }]
            }
        }
        for i, doc in enumerate(documents)
    ]
)

print(f"배치 ID: {batch.id}")
print(f"상태: {batch.processing_status}")
# 비차단 — 나중에 결과 확인

# 결과 수집 (완료 후)
for result in client.messages.batches.results(batch.id):
    if result.result.type == "succeeded":
        print(f"{result.custom_id}: {result.result.message.content[0].text}")
```

---

## Batch API vs 실시간 비교

| 항목 | Batch API | 실시간 API |
|------|----------|-----------|
| 비용 | 50% 절감 | 표준 |
| 응답 시간 | 최대 24시간 | 수 초 |
| SLA | 없음 | 있음 |
| 용도 | 비차단, 대량 | 실시간 |

---

## 야간 자동화 패턴

```python
# 매일 자정 코드베이스 분석
import schedule

def nightly_analysis():
    files = get_all_python_files()
    batch_id = create_analysis_batch(files)
    save_batch_id(batch_id)  # 다음 날 결과 확인용

schedule.every().day.at("00:00").do(nightly_analysis)
```

---

> 🔗 다음: [14.2 멀티패스 리뷰 설계](14_2_multi_pass.md)
