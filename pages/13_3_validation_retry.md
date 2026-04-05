# 13.3 검증과 재시도 루프

> 📅 2026년 04월 05일 기준

---

## 검증-재시도 패턴

```python
def extract_with_retry(document: str, max_retries: int = 3) -> dict:
    """추출 + 검증 + 재시도"""
    
    for attempt in range(max_retries):
        # 1. 추출 시도
        result = extract_with_tool_use(document)
        
        # 2. 검증
        errors = validate(result)
        
        if not errors:
            return result  # 성공!
        
        # 3. 오류 피드백으로 재시도
        print(f"시도 {attempt+1} 실패: {errors}")
        
        # 오류를 문서와 함께 다시 시도
        document = f"""
이전 추출 오류:
{errors}

위 오류를 수정하여 다시 추출하세요:
{document}
"""
    
    # 최대 재시도 후 부분 결과
    return {"data": result, "validation_errors": errors}
```

---

## 검증 레이어 설계

```python
def validate(data: dict) -> list[str]:
    """구조적 + 의미적 검증"""
    
    errors = []
    
    # 구조적 검증 (tool_use가 대부분 처리하지만 추가 확인)
    if not data.get("invoice_number"):
        errors.append("invoice_number 필수")
    
    # 의미적 검증 (tool_use가 보장하지 않음)
    amount = data.get("total_amount")
    if amount is not None and amount < 0:
        errors.append(f"total_amount 음수 불가: {amount}")
    
    due_date = data.get("due_date")
    if due_date:
        try:
            datetime.strptime(due_date, "%Y-%m-%d")
        except ValueError:
            errors.append(f"due_date 형식 오류: {due_date}")
    
    # 비즈니스 규칙 검증
    tax = data.get("tax_rate")
    if tax is not None and (tax < 0 or tax > 100):
        errors.append(f"tax_rate 범위 오류: {tax}% (0-100)")
    
    return errors
```

---

## 재시도 전략

```
재시도 가능:
- 날짜 형식 오류 (피드백으로 수정 가능)
- 금액 형식 오류 (다시 시도하면 수정 가능)

재시도 불필요:
- 필수 정보가 문서에 없음 (찾을 수 없음)
- 문서 자체가 손상됨

최대 재시도: 3회 (기본값)
```

---

> 🔗 다음: [Chapter 14: 배치 처리와 리뷰 아키텍처](14_batch_review.md)
