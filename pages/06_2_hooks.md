# 6.2 Agent SDK Hooks

> 📅 2026년 04월 05일 기준  
> ⭐ 시험 빈출 주제

---

## Hooks 개요

Hooks는 툴 실행 전후에 코드를 삽입하는 메커니즘입니다.

```
사용자 요청
    ↓
Claude (툴 호출 결정)
    ↓
PreToolUse Hook     ← 여기서 차단/수정 가능
    ↓
실제 툴 실행
    ↓
PostToolUse Hook    ← 여기서 결과 변환/정규화
    ↓
Claude (결과로 다음 결정)
```

---

## PostToolUse Hook — 결과 정규화

```python
# PostToolUse: 툴 실행 후 결과를 변환/정규화
def post_tool_use_hook(tool_name: str, raw_result: dict) -> dict:
    """툴 결과를 정규화된 형식으로 변환"""
    
    if tool_name == "search_web":
        # 외부 API 형식 → 표준 형식으로 변환
        return {
            "results": [
                {
                    "title": r.get("title", ""),
                    "url": r.get("link", ""),      # "link" → "url"
                    "snippet": r.get("description", "")
                }
                for r in raw_result.get("items", [])
            ],
            "total_results": raw_result.get("totalResults", 0)
        }
    
    elif tool_name == "get_customer":
        # 날짜 형식 정규화
        if "created_at" in raw_result:
            raw_result["created_at"] = normalize_date(raw_result["created_at"])
        return raw_result
    
    return raw_result
```

---

## PreToolUse Hook — 정책 강제

```python
# PreToolUse: 툴 실행 전 차단 또는 수정
def pre_tool_use_hook(tool_name: str, tool_input: dict) -> dict | None:
    """툴 호출 전 정책 검사
    
    Returns:
        None: 툴 실행 허용
        dict: 차단 (오류 응답 반환)
    """
    
    # 프로덕션 DB 직접 수정 차단
    if tool_name == "execute_sql":
        query = tool_input.get("query", "").upper()
        if any(keyword in query for keyword in ["DROP", "DELETE", "TRUNCATE"]):
            return {
                "isError": True,
                "errorCategory": "permission",
                "isRetryable": False,
                "message": "위험한 SQL 쿼리는 직접 실행할 수 없습니다"
            }
    
    # 고액 거래 차단
    if tool_name == "process_payment":
        amount = tool_input.get("amount", 0)
        if amount > 10000:
            return {
                "isError": True,
                "errorCategory": "business",
                "isRetryable": False,
                "message": f"$10,000 초과 거래는 수동 승인이 필요합니다"
            }
    
    return None  # 실행 허용
```

---

## Hooks 등록

```python
class AgentWithHooks:
    def __init__(self):
        self.pre_hooks = {}
        self.post_hooks = {}
    
    def register_pre_hook(self, tool_name: str, hook_fn):
        self.pre_hooks[tool_name] = hook_fn
    
    def register_post_hook(self, tool_name: str, hook_fn):
        self.post_hooks[tool_name] = hook_fn
    
    def execute_tool(self, tool_name: str, tool_input: dict) -> dict:
        # PreToolUse
        if tool_name in self.pre_hooks:
            block = self.pre_hooks[tool_name](tool_name, tool_input)
            if block:
                return block
        
        # 실제 툴 실행
        result = self._execute_actual_tool(tool_name, tool_input)
        
        # PostToolUse
        if tool_name in self.post_hooks:
            result = self.post_hooks[tool_name](tool_name, result)
        
        return result
```

---

## 핵심 정리

| Hook | 실행 시점 | 주요 용도 |
|------|---------|---------|
| PreToolUse | 툴 실행 전 | 차단, 정책 강제, 입력 검증 |
| PostToolUse | 툴 실행 후 | 결과 정규화, 형식 변환, 데이터 보강 |

---

> 🔗 다음: [6.3 태스크 분해 전략](06_3_task_decomposition.md)
