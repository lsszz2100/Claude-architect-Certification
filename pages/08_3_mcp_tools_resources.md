# 08.3 MCP 툴과 리소스 설계

> 📅 2026년 04월 05일 기준

---

## Tools vs Resources

```
Tools:     Claude가 호출하여 작업을 수행
Resources: Claude가 읽어서 정보를 얻음

Tools:
- search_issues("bug")     → 이슈 검색 결과 반환
- create_ticket({...})     → 티켓 생성
- send_message("안녕")     → 메시지 전송

Resources:
- README.md               → 파일 내용 읽기
- database://users/123    → 데이터베이스 레코드
- config://settings       → 설정 값
```

---

## MCP Tool 정의

```python
from mcp import MCPServer, tool

server = MCPServer("my-tools")

@tool(
    name="search_github_issues",
    description="""GitHub 이슈를 검색합니다.
    
    사용 시점: 버그 보고서, 기능 요청 찾기
    
    입력:
    - query: 검색어 (예: "authentication error")
    - state: "open" | "closed" | "all" (기본: "open")
    - labels: 레이블 목록 (선택)
    
    반환:
    - issues: [{number, title, body, state, labels, url}]
    - total_count: 전체 결과 수
    
    create_github_issue와 구별: 이 툴은 읽기 전용"""
)
async def search_github_issues(query: str, state: str = "open", labels: list = None):
    # 구현
    pass
```

---

## MCP Resource 정의

```python
from mcp import resource

@resource("file://{path}")
async def read_file(path: str) -> str:
    """로컬 파일 읽기"""
    with open(path) as f:
        return f.read()

@resource("db://users/{user_id}")
async def get_user(user_id: str) -> dict:
    """데이터베이스에서 사용자 정보 읽기"""
    return await db.fetch_user(user_id)
```

---

## 좋은 MCP Tool 설계 원칙

```
1. 명확한 이름: search_github_issues (동사_목적어)
2. 상세한 설명: 비슷한 툴과 구분 포함
3. 단일 책임: 한 가지 작업만
4. 에러 처리: isError, errorCategory 포함
5. 안전성: 읽기/쓰기 명확히 구분
```

---

> 🔗 다음: [08.4 내장 툴 선택 기준](08_4_builtin_tools.md)
