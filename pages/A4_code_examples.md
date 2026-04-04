# 부록 D: 실습 코드 모음

> 📅 2026년 04월 05일 기준  
> 💻 바로 실행 가능한 핵심 코드 패턴


[← 부록 C](A3_resources.md) | [목차](../TOC.md)

---

## 1. 기본 에이전틱 루프

```python
import anthropic
import json

client = anthropic.Anthropic()

def run_agent(user_message: str, tools: list) -> str:
    """기본 에이전틱 루프 — 복사해서 바로 사용 가능"""
    
    messages = [{"role": "user", "content": user_message}]
    
    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=tools,
            messages=messages
        )
        
        if response.stop_reason == "end_turn":  # ✅ 올바른 종료
            for block in response.content:
                if hasattr(block, "text"):
                    return block.text
            return "완료"
        
        elif response.stop_reason == "tool_use":
            messages.append({"role": "assistant", "content": response.content})
            
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    result = execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": json.dumps(result, ensure_ascii=False)
                    })
            
            messages.append({"role": "user", "content": tool_results})
        
        else:
            break
    
    return "예기치 않은 종료"
```

---

## 2. 프로그래밍적 게이트

```python
class WorkflowGate:
    """프로그래밍적 강제 패턴"""
    
    def __init__(self):
        self.customer_verified = False
        self.customer_id = None
    
    def execute_tool(self, tool_name: str, tool_input: dict) -> dict:
        
        # 게이트: 고객 확인 없이 환불 차단
        if tool_name == "process_refund":
            if not self.customer_verified:
                return {
                    "isError": True,
                    "errorCategory": "validation",
                    "isRetryable": False,
                    "message": "get_customer로 고객 확인 필요"
                }
        
        # 게이트: 금액 한도 확인
        if tool_name == "process_refund":
            amount = tool_input.get("amount", 0)
            if amount > 500:
                return {
                    "isError": True,
                    "errorCategory": "business",
                    "isRetryable": False,
                    "message": f"$500 초과 환불은 수동 처리 필요"
                }
        
        # 실제 툴 실행
        if tool_name == "get_customer":
            result = self._get_customer(tool_input)
            if result.get("customer_id"):
                self.customer_verified = True
                self.customer_id = result["customer_id"]
            return result
        
        return {"error": f"Unknown tool: {tool_name}"}
```

---

## 3. tool_use로 구조화된 데이터 추출

```python
def extract_invoice_data(document_text: str) -> dict:
    """tool_use로 구조화된 데이터 추출 — 환각 방지"""
    
    EXTRACTION_TOOL = {
        "name": "extract_invoice",
        "description": "인보이스에서 데이터를 추출합니다. 없는 정보는 null.",
        "input_schema": {
            "type": "object",
            "properties": {
                "invoice_number": {"type": "string"},
                "vendor_name": {"type": "string"},
                "total_amount": {"type": "number"},
                "tax_rate": {
                    "type": ["number", "null"],
                    "description": "없으면 null"
                },
                "due_date": {
                    "type": ["string", "null"],
                    "description": "YYYY-MM-DD 형식, 없으면 null"
                }
            },
            "required": ["invoice_number", "vendor_name", "total_amount",
                        "tax_rate", "due_date"]
        }
    }
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        tools=[EXTRACTION_TOOL],
        tool_choice={"type": "tool", "name": "extract_invoice"},
        messages=[{
            "role": "user",
            "content": f"추출하세요. 없는 값은 null:\n{document_text}"
        }]
    )
    
    for block in response.content:
        if block.type == "tool_use":
            return block.input
    
    return {}
```

---

## 4. PostToolUse Hook

```python
def post_tool_use_normalizer(tool_name: str, raw_result: dict) -> dict:
    """툴 결과를 정규화하는 PostToolUse Hook"""
    
    if tool_name == "get_customer":
        return {
            "customer_id": raw_result.get("id") or raw_result.get("customer_id"),
            "name": raw_result.get("full_name") or raw_result.get("name"),
            "tier": raw_result.get("membership_tier", "standard"),
            "status": raw_result.get("account_status", "active")
        }
    
    return raw_result
```

---

## 5. Batch API 활용

```python
def batch_analyze(texts: list[str]) -> str:
    """Batch API로 대량 텍스트 분석"""
    
    batch = client.messages.batches.create(
        requests=[
            {
                "custom_id": f"text-{i}",
                "params": {
                    "model": "claude-haiku-4-5-20251001",
                    "max_tokens": 256,
                    "messages": [{
                        "role": "user",
                        "content": f"분류 (긍정/부정/중립): {text}"
                    }]
                }
            }
            for i, text in enumerate(texts)
        ]
    )
    
    return batch.id  # 비차단, 나중에 결과 확인
```

---

## 6. 멀티에이전트 병렬 실행

```python
from concurrent.futures import ThreadPoolExecutor

def run_parallel_subagents(tasks: list[dict]) -> list[str]:
    """여러 서브에이전트 병렬 실행"""
    
    def run_single_agent(task: dict) -> str:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            system=f"컨텍스트: {json.dumps(task['context'])}",
            messages=[{"role": "user", "content": task["description"]}]
        )
        return response.content[0].text
    
    with ThreadPoolExecutor(max_workers=len(tasks)) as executor:
        futures = [executor.submit(run_single_agent, task) for task in tasks]
        return [f.result() for f in futures]
```

---

## 7. .mcp.json 템플릿

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "github-mcp-server",
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}",
        "GITHUB_ORG": "${GITHUB_ORG}"
      }
    },
    "postgres": {
      "type": "stdio",
      "command": "postgres-mcp-server",
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

---

## 8. CI/CD 통합

```bash
#!/bin/bash
# ci-review.sh

RESULT=$(claude -p "
변경된 파일의 보안 취약점을 분석하세요:
$(git diff --name-only origin/main...HEAD)

JSON으로 응답하세요:
{\"score\": 0-100, \"issues\": [], \"passed\": true/false}
" --output-format json)

PASSED=$(echo "$RESULT" | jq '.passed')

if [ "$PASSED" = "false" ]; then
    echo "❌ 코드 리뷰 실패"
    echo "$RESULT" | jq '.issues[]'
    exit 1
fi

echo "✅ 코드 리뷰 통과"
```

---

> 🔗 이 책의 처음으로: [이 책을 읽는 방법](00_how_to_read.md)
