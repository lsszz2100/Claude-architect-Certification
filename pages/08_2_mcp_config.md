# 8.2 MCP 서버 설정과 구성

> 📅 2026년 04월 05일 기준  
> ⭐ 시험 핵심 — 파일 위치 암기 필수

---

## 설정 파일 위치

| 파일 | 범위 | 특징 |
|------|------|------|
| `.mcp.json` (프로젝트 루트) | 팀 공유 | 버전 관리, 모든 팀원 |
| `~/.claude.json` (사용자 홈) | 개인 전용 | 버전 관리 안 됨 |

---

## .mcp.json 구조

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "github-mcp-server",
      "args": ["--verbose"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}",
        "GITHUB_ORG": "my-company"
      }
    },
    "postgres": {
      "type": "stdio",
      "command": "postgres-mcp-server",
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    },
    "slack": {
      "type": "sse",
      "url": "https://slack-mcp.example.com/sse",
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}"
      }
    }
  }
}
```

---

## 환경 변수 참조 (${변수명})

```json
{
  "env": {
    "API_KEY": "${MY_API_KEY}"
  }
}
```

이유: API 키를 파일에 하드코딩하면 버전 관리 시 유출 위험

```bash
# 개발자 개인 환경에서 설정
export MY_API_KEY="sk-..."

# CI/CD에서는 시크릿으로 설정
# GitHub Actions: secrets.MY_API_KEY
```

---

## MCP 서버 유형

```
stdio: 로컬 프로세스로 실행
  command: 실행할 명령어
  args: 추가 인수

sse: Server-Sent Events (원격 서버)
  url: SSE 엔드포인트 URL
```

---

## 개인 MCP 설정

```json
// ~/.claude.json (개인 설정)
{
  "mcpServers": {
    "personal-notes": {
      "type": "stdio",
      "command": "notes-mcp",
      "env": {
        "NOTES_PATH": "${HOME}/notes"
      }
    }
  }
}
```

---

> 🔗 다음: [8.3 MCP 툴과 리소스 설계](08_3_mcp_tools_resources.md)
