# 08.1 MCP란 무엇인가?

> 📅 2026년 04월 05일 기준

---

## Model Context Protocol (MCP)

```
MCP = Claude와 외부 시스템을 연결하는 표준 프로토콜

없이:   Claude ←독자 방식→ 각 서비스 (매번 다른 통합)
있으면: Claude ←MCP→ 모든 호환 서비스 (표준화된 방식)
```

---

## MCP가 해결하는 문제

```
기존 방식:
GitHub 통합 → 자체 SDK 사용
Jira 통합   → 자체 REST API 코드
Slack 통합  → 자체 Webhook 구현
→ 각각 다른 방식으로 개발, 유지 비용 높음

MCP 방식:
GitHub MCP 서버 → 표준 MCP 인터페이스
Jira MCP 서버   → 표준 MCP 인터페이스
Slack MCP 서버  → 표준 MCP 인터페이스
→ 동일한 방식으로 모든 서비스 연결
```

---

## MCP 구성 요소

```
MCP 서버:     외부 서비스를 MCP로 노출
MCP 클라이언트: Claude Code, SDK 등
MCP 프로토콜:  서버-클라이언트 통신 표준
```

---

## MCP가 제공하는 것

```
1. Tools: Claude가 호출할 수 있는 기능
   예: search_github_issues, create_jira_ticket

2. Resources: Claude가 읽을 수 있는 데이터
   예: 파일, 데이터베이스 레코드

3. Prompts: 재사용 가능한 프롬프트 템플릿
```

---

## 실제 MCP 활용 사례

```
개발자 도구:
- GitHub MCP 서버: PR 조회, 코드 검색
- Jira MCP 서버: 티켓 관리
- Slack MCP 서버: 메시지 전송

비즈니스 도구:
- Salesforce MCP: CRM 데이터
- PostgreSQL MCP: 데이터베이스 쿼리
- Google Drive MCP: 문서 접근
```

---

> 🔗 다음: [08.2 MCP 서버 설정과 구성](08_2_mcp_config.md)
