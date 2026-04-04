# 13.2 스키마 설계 원칙

> 📅 2026년 04월 05일 기준

---

## 핵심 원칙: nullable 필드

```python
# ❌ 잘못된 설계 — 없는 값을 추측하게 만듦
{
    "properties": {
        "tax_rate": {"type": "number"}  # null 불가
    },
    "required": ["tax_rate"]
}
# 세율이 없으면 LLM이 임의로 값 생성 (환각!)

# ✅ 올바른 설계 — null 명시 허용
{
    "properties": {
        "tax_rate": {
            "type": ["number", "null"],
            "description": "세율 %. 문서에 명시되지 않은 경우 null"
        }
    },
    "required": ["tax_rate"]
}
```

---

## 스키마 설계 체크리스트

```
□ 문서에 항상 있는 필드만 required에 포함
□ 선택적 필드는 nullable (["type", "null"])
□ 설명에 "없으면 null" 명시
□ 날짜는 ISO 8601 형식 지정 (YYYY-MM-DD)
□ 금액은 숫자 타입 (통화 기호 제거 지시)
□ enum으로 허용 값 제한
```

---

## 완전한 스키마 예시

```python
INVOICE_SCHEMA = {
    "type": "object",
    "properties": {
        # 필수 — 항상 존재
        "invoice_number": {
            "type": "string",
            "description": "인보이스 번호 (예: INV-2024-001)"
        },
        "vendor_name": {
            "type": "string",
            "description": "발행 회사명"
        },
        "total_amount": {
            "type": "number",
            "description": "총 금액 (통화 기호 제외, 숫자만)"
        },
        # 선택적 — nullable
        "tax_rate": {
            "type": ["number", "null"],
            "description": "세율 %. 명시되지 않으면 null"
        },
        "payment_terms": {
            "type": ["string", "null"],
            "enum": ["net30", "net60", "immediate", None],
            "description": "지불 조건. 명시되지 않으면 null"
        },
        "due_date": {
            "type": ["string", "null"],
            "pattern": "^\\d{4}-\\d{2}-\\d{2}$",
            "description": "지불 기한 YYYY-MM-DD. 없으면 null"
        }
    },
    "required": [
        "invoice_number", "vendor_name", "total_amount",
        "tax_rate", "payment_terms", "due_date"
    ]
}
```

---

> 🔗 다음: [13.3 검증과 재시도 루프](13_3_validation_retry.md)
