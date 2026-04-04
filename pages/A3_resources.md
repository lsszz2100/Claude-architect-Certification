# 부록 C: 추가 학습 리소스

> 📅 2026년 04월 05일 기준  
> 📚 공식 자료 및 추가 학습 자료 모음

---

## 공식 Anthropic 자료

### 시험 관련
- 시험 등록: https://anthropic.skilljar.com
- 공식 강의 (13개 무료): https://anthropic.skilljar.com
- 연습 문제: http://claudecertifications.com

### 개발 문서
- API 문서: https://docs.anthropic.com
- Claude Code 문서: https://docs.anthropic.com/claude-code
- MCP 문서: https://modelcontextprotocol.io

### 실습 코드
- Anthropic Cookbook: https://github.com/anthropics/anthropic-cookbook
  - 에이전트 패턴
  - MCP 예제
  - 프롬프트 엔지니어링 가이드

---

## 도메인별 핵심 학습 자료

### Domain 1: Agentic Architecture

필독 문서:
- Building effective agents (Anthropic 공식 가이드)
- Multi-agent systems patterns

실습 포인트:
- 에이전틱 루프 직접 구현
- Hub-and-Spoke 패턴으로 3+ 에이전트 시스템 구축

### Domain 2: Tool Design & MCP

필독 문서:
- Tool use documentation
- MCP specification

실습 포인트:
- 좋은 툴 설명 10개 직접 작성
- MCP 서버 설정 실습

### Domain 3: Claude Code

필독 문서:
- Claude Code documentation
- CLAUDE.md guide

실습 포인트:
- 실제 프로젝트에 CLAUDE.md 적용
- 팀 커맨드 만들어보기
- -p 플래그로 CI/CD 연습

### Domain 4: Prompt Engineering

필독 문서:
- Prompt engineering guide
- Structured output guide

실습 포인트:
- 명시적 기준 작성 연습
- Few-shot 예시 만들기
- tool_use로 JSON 추출

### Domain 5: Context Management

실습 포인트:
- 요약 지시문 작성 (수치 보존)
- 에스컬레이션 기준 프롬프트 작성
- 상충 정보 처리 패턴

---

## 추천 학습 순서

```
1단계 (입문): API 기초 + 첫 에이전트 구축 (2주)
2단계 (중급): 멀티에이전트 + 툴 설계 (3주)
3단계 (고급): Claude Code + 프롬프트 엔지니어링 (3주)
4단계 (실전): 시나리오 실습 + 문제 풀이 (4주)
```

---

## 실습 환경 설정

```bash
# 필수 도구
pip install anthropic python-dotenv

# Claude Code (선택)
npm install -g @anthropic-ai/claude-code

# 환경 변수
export ANTHROPIC_API_KEY="sk-ant-..."
```

---

## 커뮤니티 및 지원

- Anthropic 공식 Discord: 개발자 커뮤니티
- GitHub Issues: https://github.com/anthropics/claude-code/issues
- Stack Overflow: claude-api 태그

---

> 🔗 관련: [부록 D: 실습 코드 모음](A4_code_examples.md)
