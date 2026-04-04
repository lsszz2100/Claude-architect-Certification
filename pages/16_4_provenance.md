# 16.4 정보 출처 보존

> 📅 2026년 04월 05일 기준  
> ⭐ 시험 핵심 — 상충 정보 처리

---

## 정보 출처 보존이 중요한 이유

```
두 소스에서 다른 정보가 올 때:
소스 A: "가격 $99.99"
소스 B: "가격 $89.99"

❌ 임의 선택: 둘 중 하나를 선택하여 보고
✅ 출처 보존: 두 값을 모두 출처와 함께 보고
```

---

## 상충 정보 처리 패턴

```python
def handle_conflicting_info(sources: list[dict]) -> dict:
    """상충 정보를 출처와 함께 보고"""
    
    consolidated = {}
    conflicts = []
    
    for field in get_all_fields(sources):
        values = [(s["source"], s["data"].get(field)) for s in sources]
        
        # 모든 소스가 동의하는 경우
        unique_values = set(v for _, v in values if v is not None)
        
        if len(unique_values) == 1:
            consolidated[field] = list(unique_values)[0]
        
        elif len(unique_values) > 1:
            # 상충 — 모두 보고
            conflicts.append({
                "field": field,
                "values": [
                    {"source": src, "value": val}
                    for src, val in values
                ],
                "requires_human_review": True
            })
    
    return {
        "consolidated": consolidated,
        "conflicts": conflicts,
        "confidence": "low" if conflicts else "high"
    }
```

---

## 출처 추적 패턴

```python
# 모든 데이터에 출처 메타데이터 포함
def fetch_with_provenance(source_name: str, query: str) -> dict:
    data = fetch_data(query)
    
    return {
        "data": data,
        "provenance": {
            "source": source_name,
            "fetched_at": datetime.now().isoformat(),
            "query": query,
            "confidence": "high"
        }
    }
```

---

## 시험 핵심 정리

```
Q: 두 소스에서 다른 가격 정보가 오면?
A: 두 가격을 모두 출처와 함께 보고하고 인간 검토 요청

Q: 상충 정보를 임의로 선택하는 것의 문제는?
A: 어떤 정보가 맞는지 알 수 없고, 오류 원인 추적이 불가능

✅ 원칙: 불확실한 상황에서 임의 선택 금지
→ 두 값을 출처와 함께 투명하게 제공
```

---

> 🔗 다음: [Part 7: 실전 시나리오](../17_scenario1_customer_support.md)
