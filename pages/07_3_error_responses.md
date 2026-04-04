# 7.3 구조화된 에러 응답 설계

> 📅 2026년 04월 05일 기준  
> ⭐ 시험 핵심 — 에러 분류 암기 필수

---

## 에러 응답 구조

```python
# 표준 에러 응답 형식
error_response = {
    "isError": True,                    # 에러 여부
    "errorCategory": "transient",       # 에러 유형
    "isRetryable": True,                # 재시도 가능 여부
    "message": "서버 응답 시간 초과",    # 사람이 읽을 수 있는 설명
    "details": {...}                    # 추가 정보 (선택)
}
```

---

## 에러 카테고리 (암기 필수!)

| errorCategory | isRetryable | 예시 |
|--------------|-------------|------|
| transient | ✅ True | 타임아웃, 서버 오류, 네트워크 장애 |
| validation | ❌ False | 잘못된 입력 형식, 필수 필드 누락 |
| business | ❌ False | 정책 위반, 한도 초과, 권한 없음 |
| permission | ❌ False | 인증 실패, 접근 거부 |

---

## 각 에러 유형 예시

```python
# transient: 일시적 오류 — 재시도하면 해결될 수 있음
{"isError": True, "errorCategory": "transient", "isRetryable": True,
 "message": "데이터베이스 연결 실패. 잠시 후 재시도하세요."}

# validation: 입력 문제 — 입력을 수정해야 함
{"isError": True, "errorCategory": "validation", "isRetryable": False,
 "message": "이메일 형식이 올바르지 않습니다. 예: user@example.com"}

# business: 비즈니스 규칙 위반 — 정책 변경 없이는 해결 불가
{"isError": True, "errorCategory": "business", "isRetryable": False,
 "message": "자동 환불 한도($500) 초과. 수동 처리 필요."}

# permission: 권한 없음 — 권한 부여 없이는 해결 불가
{"isError": True, "errorCategory": "permission", "isRetryable": False,
 "message": "이 고객 계정에 접근 권한이 없습니다."}
```

---

## 빈 결과 vs 에러 구분

```python
# ❌ 잘못된 방법: 빈 결과를 에러로 처리
if not results:
    return {"isError": True, "message": "결과 없음"}

# ✅ 올바른 방법: 빈 결과와 에러를 구분
def search_orders(customer_id: str):
    try:
        results = db.query(customer_id)
        
        if results == []:
            # 결과 없음 — 에러 아님
            return {"orders": [], "total": 0, "message": "주문 내역 없음"}
        
        return {"orders": results, "total": len(results)}
    
    except DatabaseConnectionError:
        # 접근 실패 — 에러
        return {
            "isError": True,
            "errorCategory": "transient",
            "isRetryable": True,
            "message": "데이터베이스 접근 실패"
        }
```

---

> 🔗 다음: [7.4 tool_choice 사용법](07_4_tool_choice.md)
