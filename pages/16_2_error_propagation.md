# 16.2 에러 전파 전략

> 📅 2026년 04월 05일 기준

---

## 구조화된 에러 컨텍스트

단순히 오류 메시지만 전파하면 다음 에이전트/단계가 상황을 이해하기 어렵습니다.

```python
# ❌ 단순한 오류 전파
raise Exception("데이터베이스 오류")

# ✅ 구조화된 오류 컨텍스트
return {
    "errorType": "database_timeout",
    "attemptedQuery": "SELECT * FROM orders WHERE customer_id = 123",
    "partialResults": [{"order_id": "ORD-001"}, {"order_id": "ORD-002"}],
    "isRetryable": True,
    "retryAfterSeconds": 30,
    "suggestedAlternatives": [
        "캐시된 데이터 사용",
        "부분 결과로 계속 진행"
    ]
}
```

---

## 에러 전파 체인

```
서브에이전트 → 코디네이터 → 사용자

각 단계에서:
1. 오류 유형 분류
2. 부분 결과 보존
3. 재시도 가능 여부 판단
4. 대안 제안
```

---

## 멀티에이전트에서 에러 처리

```python
def handle_subagent_error(error: dict, partial_results: list) -> dict:
    """서브에이전트 오류를 코디네이터가 처리"""
    
    if error["isRetryable"]:
        # 재시도
        return retry_subagent(error["attemptedTask"])
    
    elif error.get("partialResults"):
        # 부분 결과로 계속
        return {
            "status": "partial",
            "data": error["partialResults"],
            "warning": f"일부 데이터 누락: {error['errorType']}"
        }
    
    else:
        # 대안 사용
        for alternative in error.get("suggestedAlternatives", []):
            result = try_alternative(alternative)
            if result:
                return result
        
        # 최후 수단: 에스컬레이션
        return escalate_to_human(error)
```

---

## 에러 컨텍스트 보존

```
에러 전파 시 반드시 포함:
✅ 오류 유형 (errorType)
✅ 시도된 작업 (attemptedQuery/Task)
✅ 부분 결과 (partialResults)
✅ 재시도 가능 여부 (isRetryable)
✅ 대안 방법 (suggestedAlternatives)

❌ 포함하면 안 되는 것:
- 스택 트레이스 전체 (민감한 내부 정보)
- 데이터베이스 연결 문자열
```

---

> 🔗 다음: [16.3 대규모 코드베이스 탐색](16_3_large_codebase.md)
