# 4.3 stop_reason 이해하기

> 📅 2026년 04월 05일 기준  
> ⭐ 시험 빈출 주제

---

## stop_reason 값 목록

| stop_reason | 의미 | 루프 처리 |
|-------------|------|----------|
| `"end_turn"` | 모델이 완료 신호 | 루프 종료 ✅ |
| `"tool_use"` | 툴 호출 필요 | 툴 실행 후 계속 |
| `"max_tokens"` | 토큰 한계 도달 | 처리 필요 (오류 or 계속) |
| `"stop_sequence"` | 지정한 중단 시퀀스 감지 | 조건에 따라 처리 |

---

## 각 stop_reason 처리 방법

```python
if response.stop_reason == "end_turn":
    # ✅ 정상 완료 — 루프 종료
    return extract_text(response)

elif response.stop_reason == "tool_use":
    # 툴 실행 후 계속
    results = execute_all_tools(response)
    messages.extend(results)
    continue

elif response.stop_reason == "max_tokens":
    # 토큰 한계 — 처리 필요
    # 옵션 1: 오류 반환
    # 옵션 2: 더 긴 max_tokens로 재시도
    # 옵션 3: 현재까지 결과 부분 반환
    handle_token_limit(response)

elif response.stop_reason == "stop_sequence":
    # 지정한 시퀀스 감지 — 맥락에 따라 처리
    handle_stop_sequence(response)
```

---

## ❌ 절대 하면 안 되는 패턴

```python
# ❌ 텍스트로 종료 판단
if "DONE" in response.content[0].text:
    break

# ❌ "TASK COMPLETE" 찾기
if "완료" in response_text:
    break

# ❌ 고정 횟수만으로 종료 (안전망으로는 OK)
for i in range(10):  # ← 단독으로 사용 금지
    ...
```

왜 안 되는가?
- LLM은 다양한 맥락에서 "DONE"을 생성할 수 있음
- 텍스트 기반 감지는 신뢰할 수 없음
- `stop_reason`만이 모델의 공식 완료 신호

---

## 시험 핵심 정리

```
✅ "end_turn" = 루프 종료 (정상 완료)
✅ "tool_use" = 툴 실행 후 계속
❌ 텍스트로 종료 판단 금지
❌ 고정 횟수만으로 종료 금지 (안전망으로만 사용)
```

---

> 🔗 다음: [Chapter 5: 멀티에이전트 시스템 설계](../05_multi_agent.md)
