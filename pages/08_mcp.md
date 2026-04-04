# Chapter 8: Model Context Protocol (MCP)

> 📅 2026년 04월 05일 기준  
> 🎯 Domain 2 핵심 — MCP 설정과 리소스 설계

---

## 8.1 MCP란 무엇인가?

### USB-C 비유

> MCP는 AI 애플리케이션을 위한 USB-C 포트와 같습니다.

USB-C가 하나의 표준 포트로 다양한 기기를 연결하듯, MCP는 하나의 표준 프로토콜로 Claude를 다양한 외부 시스템에 연결합니다.

```
MCP 없이:                    MCP 있이:
각 AI마다 다른 연결 방법       표준화된 하나의 방법

Claude ─── 직접 API ──→ Jira    Claude ─── MCP ──→ Jira MCP Server
Claude ─── 직접 API ──→ GitHub  Claude ─── MCP ──→ GitHub MCP Server
Claude ─── 직접 API ──→ Slack   Claude ─── MCP ──→ Slack MCP Server
```

### MCP의 세 가지 구성 요소

| 구성 요소 | 역할 | 예시 |
|----------|------|------|
| Tools | 액션 수행 | 파일 읽기, API 호출, DB 쿼리 |
| Resources | 콘텐츠 노출 | 이슈 목록, 문서 카탈로그, 스키마 |
| Prompts | 워크플로우 템플릿 | 코드 리뷰 프롬프트, 분석 템플릿 |

---

## 8.2 MCP 서버 설정과 구성

> 🎯 시험 출제: 프로젝트 범위 vs 사용자 범위

### MCP 서버 범위

| 범위 | 파일 위치 | 공유 여부 | 용도 |
|------|----------|----------|------|
| 프로젝트 범위 | `.mcp.json` (프로젝트 루트) | ✅ 팀 전체 공유 (버전 관리) | 팀 공통 툴링 |
| 사용자 범위 | `~/.claude.json` | ❌ 개인만 사용 | 개인 실험용 |

### .mcp.json 설정 예시

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "jira": {
      "command": "python",
      "args": ["-m", "jira_mcp_server"],
      "env": {
        "JIRA_URL": "${JIRA_URL}",
        "JIRA_TOKEN": "${JIRA_API_TOKEN}"
      }
    },
    "custom-crm": {
      "command": "./scripts/crm_mcp_server.py",
      "env": {
        "CRM_API_KEY": "${CRM_API_KEY}",
        "CRM_BASE_URL": "${CRM_BASE_URL}"
      }
    }
  }
}
```

> ⚠️ `${변수명}` 형식으로 환경 변수를 참조하세요!  
> API 키를 .mcp.json에 직접 하드코딩하면 Git에 노출될 위험이 있습니다.

### 개인용 MCP 서버 (~/.claude.json)

```json
{
  "mcpServers": {
    "my-experimental-tool": {
      "command": "python",
      "args": ["/home/user/experiments/my_mcp_server.py"],
      "env": {
        "DEBUG": "true"
      }
    }
  }
}
```

---

## 8.3 MCP 툴과 리소스 설계

### MCP 리소스 활용

> 🎯 리소스 = 탐색적 툴 호출 감소

```python
# ❌ 비효율적: 에이전트가 일일이 탐색해야 함
# "어떤 이슈가 있지? → Jira 검색 → 또 검색 → 또 검색..."

# ✅ 효율적: 리소스로 카탈로그 제공
# MCP 서버가 이슈 요약 리소스를 제공:

class JiraMCPServer:
    def get_resources(self):
        return [
            {
                "uri": "jira://issues/summary",
                "name": "현재 스프린트 이슈 요약",
                "description": "현재 스프린트의 모든 이슈 목록과 상태",
                "mimeType": "application/json"
            },
            {
                "uri": "jira://project/schema",
                "name": "프로젝트 스키마",
                "description": "이슈 유형, 상태, 우선순위 정의",
                "mimeType": "application/json"
            }
        ]
```

### 커뮤니티 MCP vs 커스텀 MCP

> 🎯 표준 통합에는 커뮤니티 MCP를 먼저 사용!

```python
# 의사결정 기준:
# 1. 표준 도구 (Jira, GitHub, Slack 등) → 커뮤니티 MCP 사용
# 2. 사내 특화 시스템 → 커스텀 MCP 개발

# ✅ Jira는 커뮤니티 MCP 사용
# @modelcontextprotocol/server-jira 사용

# ✅ 사내 CRM은 커스텀 개발
# ./crm_mcp_server.py 직접 작성
```

---

## 8.4 내장 툴 선택 기준

> 🎯 시험 출제: Grep vs Glob vs Read vs Edit

### 내장 툴 선택 가이드

| 작업 | 사용할 툴 | 이유 |
|------|----------|------|
| 코드에서 패턴 검색 | Grep | 파일 내용을 패턴으로 검색 |
| 이름 패턴으로 파일 찾기 | Glob | 파일 경로 패턴 매칭 |
| 파일 전체 읽기 | Read | 전체 파일 내용 로드 |
| 특정 부분만 수정 | Edit | 유니크 텍스트 매칭으로 교체 |
| 파일 전체 교체 | Write | 전체 파일 재작성 |
| 명령 실행 | Bash | 쉘 명령 실행 |

### 실전 선택 예시

```python
# 🔍 함수 사용처 찾기 → Grep
grep_result = grep(pattern="def process_payment", path="./src")
# → 파일 내용 검색

# 📁 테스트 파일 모두 찾기 → Glob
test_files = glob(pattern="**/*.test.tsx")
# → 파일 경로 패턴 검색

# 📖 파일 내용 분석 → Read
content = read(path="./src/payment/processor.py")
# → 전체 파일 읽기

# ✏️ 특정 함수 수정 → Edit
edit(
    path="./src/payment/processor.py",
    old_string="def process_payment(amount):",
    new_string="def process_payment(amount: float) -> dict:"
)
# → Edit 실패 시 (비유니크) → Read + Write 조합 사용
```

### 코드베이스 탐색 전략

```
✅ 효율적인 탐색 순서:
1. Grep으로 진입점 찾기 (예: "class PaymentProcessor")
2. Read로 해당 파일 읽어 임포트 추적
3. Glob으로 관련 파일 패턴 찾기
4. Read로 관련 파일들 순차 분석

❌ 비효율적: 처음부터 모든 파일 읽기
```

---

## 📝 챕터 요약

- MCP = AI와 외부 시스템 연결의 표준 프로토콜
- 프로젝트 범위 (`.mcp.json`): 팀 공유 / 사용자 범위 (`~/.claude.json`): 개인용
- 환경 변수 `${변수명}`으로 시크릿 관리
- 리소스로 콘텐츠 카탈로그 제공 → 탐색 비용 절감
- 표준 도구는 커뮤니티 MCP 우선, 특화 시스템은 커스텀
- Grep(내용 검색) / Glob(파일 경로) / Read(전체 읽기) / Edit(부분 수정)

---

> 🔗 다음 챕터: [Claude Code 실전 활용](09_claude_code.md)
