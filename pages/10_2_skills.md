# 10.2 Skills 시스템 심화

> 📅 2026년 04월 05일 기준

---

## Skills = 고급 슬래시 커맨드

Skills는 YAML frontmatter로 동작을 세밀하게 제어할 수 있는 커맨드입니다.

---

## frontmatter 옵션 전체

```markdown
---
description: "PR 코드 리뷰 수행"      # 커맨드 설명
context: fork                          # 격리 실행 여부
allowed-tools: Read, Grep, Glob        # 허용 툴 목록
argument-hint: "PR 번호 (필수)"         # 인수 안내
---

$ARGUMENTS에 해당하는 PR을 분석하세요.
```

---

## context: fork 상세

```
context: fork:
- 현재 세션과 격리된 서브에이전트에서 실행
- 현재 세션 컨텍스트 오염 없음
- 독립적으로 실행되고 결과만 반환

언제 사용:
✅ 실험적인 코드 탐색
✅ 대규모 분석 (현재 작업에 영향 없이)
✅ 독립적인 검증 작업

언제 사용 안 함:
현재 세션과 상태를 공유해야 할 때
```

---

## allowed-tools 세밀 제어

```markdown
---
allowed-tools: Read, Grep, Glob    # 읽기만 허용 (안전)
---
코드 분석만 수행 (수정 불가)
```

```markdown
---
allowed-tools: Read, Write, Bash   # 쓰기 가능
---
파일 생성 및 수정 가능
```

---

## 실전 Skills 예시

```markdown
<!-- .claude/commands/security-audit.md -->
---
description: "보안 취약점 감사"
context: fork
allowed-tools: Read, Grep, Glob
argument-hint: "감사할 디렉토리 (기본: src/)"
---

다음 보안 취약점을 검사하세요:
- SQL 인젝션
- XSS (Cross-Site Scripting)
- 하드코딩된 자격증명
- 안전하지 않은 파일 접근

검사 대상: $ARGUMENTS (없으면 src/ 전체)

결과를 심각도별로 분류하여 보고하세요.
```

---

> 🔗 다음: [10.3 context:fork와 격리 실행](10_3_fork_context.md)
