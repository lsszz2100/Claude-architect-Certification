# 02.1 Opus, Sonnet, Haiku — 언제 무엇을 쓸까?

> 📅 2026년 04월 05일 기준

---

## 모델 비교표

| 모델 | 컨텍스트 | 입력 가격 | 출력 가격 | 특징 |
|------|---------|---------|---------|------|
| claude-opus-4-6 | 1M 토큰 | $5/1M | $25/1M | 가장 강력, 복잡한 추론 |
| claude-sonnet-4-6 | 1M 토큰 | $3/1M | $15/1M | 균형, 일반 작업 최적 |
| claude-haiku-4-5 | 200K 토큰 | $1/1M | $5/1M | 빠름, 비용 효율 |

---

## 모델 선택 기준

### Opus 04.6 사용 시나리오

```python
# 복잡한 추론이 필요한 경우
model = "claude-opus-4-6"

사용 사례:
- 다단계 연구 보고서 작성
- 복잡한 코드 아키텍처 설계
- 멀티에이전트 코디네이터
- 법률/의료 문서 분석
```

### Sonnet 04.6 사용 시나리오

```python
# 균형 잡힌 성능이 필요한 경우 (기본 선택)
model = "claude-sonnet-4-6"

사용 사례:
- 일반 고객 지원
- 코드 생성 및 리뷰
- 서브에이전트 실행
- 대부분의 프로덕션 작업
```

### Haiku 04.5 사용 시나리오

```python
# 속도와 비용이 우선인 경우
model = "claude-haiku-4-5-20251001"

사용 사례:
- 단순 분류 작업
- 빠른 요약
- 대량 배치 처리
- 실시간 응답이 필요한 가벼운 작업
```

---

## 비용 최적화 전략

```python
def select_model(task_complexity: str) -> str:
    """작업 복잡도에 따른 모델 선택"""
    
    model_map = {
        "complex": "claude-opus-4-6",      # 연구, 복잡한 추론
        "standard": "claude-sonnet-4-6",    # 일반 작업 (기본)
        "simple": "claude-haiku-4-5-20251001"  # 분류, 간단한 추출
    }
    
    return model_map.get(task_complexity, "claude-sonnet-4-6")
```

---

## 시험 팁

모델 ID 정확히 외우기:
- `claude-opus-4-6`
- `claude-sonnet-4-6`
- `claude-haiku-4-5-20251001`

---

> 🔗 다음: [02.2 컨텍스트 윈도우와 토큰 이해](02_2_context_tokens.md)
