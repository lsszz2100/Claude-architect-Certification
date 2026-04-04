# 09.1 Claude Code란?

> 📅 2026년 04월 05일 기준

---

## Claude Code 개요

Claude Code는 Claude를 개발 환경에 통합하는 공식 CLI 및 IDE 확장입니다.

```
사용 가능한 환경:
- CLI: 터미널에서 `claude` 명령어
- VS Code 확장
- JetBrains 플러그인
- 웹 앱 (claude.ai/code)
```

---

## 주요 기능

```
1. 자연어로 코드 생성 및 수정
2. 코드베이스 탐색 및 이해
3. 버그 분석 및 수정
4. 커스텀 커맨드 (/review, /test 등)
5. CI/CD 통합 (-p 플래그)
6. MCP 서버 연결
```

---

## Claude Code가 일반 Claude와 다른 점

| 기능 | 일반 Claude | Claude Code |
|------|------------|------------|
| 파일 시스템 접근 | ❌ | ✅ |
| 터미널 명령 실행 | ❌ | ✅ |
| 코드베이스 탐색 | ❌ | ✅ |
| 커스텀 커맨드 | ❌ | ✅ |
| CI/CD 통합 | ❌ | ✅ (-p 플래그) |
| CLAUDE.md 가이드라인 | ❌ | ✅ |

---

## 첫 시작

```bash
# CLI 설치
npm install -g @anthropic-ai/claude-code

# 프로젝트에서 실행
cd my-project
claude

# 비대화형 모드 (CI/CD)
claude -p "코드 리뷰를 수행하세요"
```

---

> 🔗 다음: [09.2 CLAUDE.md 설정 계층 구조](09_2_claude_md.md)
