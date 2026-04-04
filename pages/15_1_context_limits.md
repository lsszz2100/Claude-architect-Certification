# 15.1 컨텍스트 창의 한계와 대응

> 📅 2026년 04월 05일 기준

---

## 컨텍스트 한계

```
Claude Opus/Sonnet: 1,000,000 토큰 ≈ 750만 단어
Claude Haiku:       200,000 토큰  ≈ 150만 단어

현실적 제한:
- 너무 긴 컨텍스트 = 비용 증가
- Lost-in-the-Middle 효과
- 응답 속도 저하
```

---

## 컨텍스트 관리 전략

### 1. 트리밍 (Trimming)

```python
def trim_old_messages(messages: list, max_tokens: int = 800_000) -> list:
    """오래된 메시지 제거"""
    
    while calculate_tokens(messages) > max_tokens:
        if len(messages) <= 2:  # 최소 유지
            break
        # 두 번째 메시지 제거 (첫 번째는 시스템/사용자 유지)
        messages.pop(1)
    
    return messages
```

### 2. 요약 (Summarization)

```python
def summarize_history(messages: list) -> str:
    """이전 대화를 요약으로 압축"""
    
    history_text = "\n".join([
        f"{m['role']}: {m['content']}" for m in messages[:-5]
    ])
    
    summary = claude.create(
        messages=[{
            "role": "user",
            "content": f"""다음 대화를 요약하세요.
수치(금액, 날짜, ID)를 반드시 포함하세요:
{history_text}"""
        }]
    )
    
    return summary.content[0].text
```

### 3. 핵심 사실 블록

```python
# 매 턴마다 핵심 사실 포함
FACTS_BLOCK = """
[핵심 사실 — 항상 참조]
고객 ID: CUST-12345
주문 번호: ORD-67890
환불 금액: $150.00
케이스 시작: 2024-03-10
"""
```

---

## 컨텍스트 효율화 팁

```
1. 필요한 정보만 포함 (불필요한 대화 제거)
2. 핵심 사실은 구조화된 블록으로
3. 오래된 세부 사항은 요약으로 압축
4. 중요 정보는 앞뒤에 배치
```

---

> 🔗 다음: [15.2 Lost-in-the-Middle 효과](15_2_lost_middle.md)
