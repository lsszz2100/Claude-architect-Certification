# 이 책을 읽는 방법

> 📅 최종 업데이트: 2026년 04월 05일

---

## 이 책의 구성 원칙

이 책은 **"배우면서 만든다(Learn by Building)"** 는 철학으로 설계되었습니다.

단순히 개념을 설명하는 것에서 그치지 않고, **AI 업무 자동화 비서**라는 하나의 일관된 프로젝트를 통해 각 개념을 실제로 적용하는 경험을 제공합니다.

---

## 독자 수준별 권장 경로

### 🟢 완전 초보자 (AI/프로그래밍 처음)
```
Part 1 (기초) → Part 2 일부 → Part 7 시나리오 → Part 8 문제 → Part 9 전략
```
- Part 1을 충분히 읽고 API 호출을 직접 해보세요
- 개념을 100% 이해하려 하지 말고, 흐름을 파악하는 데 집중하세요

### 🟡 개발 경험 있는 입문자
```
Part 1 빠르게 → Part 2~6 전체 → Part 7 → Part 8 → Part 9
```
- API 섹션은 빠르게 넘기고 에이전트 설계에 집중하세요

### 🔴 경력 개발자 / 빠른 합격 목표
```
Part 9 전략 먼저 → Part 2~6 도메인 순서대로 → Part 8 문제 풀이 → 부록 치트시트
```

---

## 아이콘 가이드

| 아이콘 | 의미 |
|--------|------|
| 💡 | 핵심 개념 설명 |
| ⚠️ | 자주 틀리는 함정 |
| 🎯 | 시험 출제 포인트 |
| 💻 | 실습 코드 |
| 📝 | 요약 정리 |
| 🔗 | 참고 링크 |

---

## 실습 환경 준비

이 책의 예제 코드를 실행하려면:

1. **Python 3.10+** 설치
2. **Anthropic SDK** 설치:
   ```bash
   pip install anthropic
   ```
3. **API 키** 발급: [https://console.anthropic.com](https://console.anthropic.com)
4. 환경 변수 설정:
   ```bash
   export ANTHROPIC_API_KEY="your-api-key-here"
   ```

---

## 학습 자료 출처

이 책은 다음 공식 자료를 기반으로 작성되었습니다:

- **Claude Certified Architect – Foundations Exam Guide** (v0.1, 2025.02.10)
- **Anthropic 공식 문서**: https://docs.anthropic.com
- **Claude Cookbook**: https://github.com/anthropics/anthropic-cookbook
- **MCP 공식 문서**: https://modelcontextprotocol.io
- **Claude Certifications**: https://claudecertifications.com

---

> 📌 **중요**: 이 책의 모든 코드와 개념은 2026년 04월 05일 기준으로 검증되었습니다. Claude API는 지속적으로 업데이트되므로, 최신 변경사항은 공식 문서를 함께 참고하세요.
