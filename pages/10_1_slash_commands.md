# 10.1 슬래시 커맨드 만들기

> 📅 2026년 04월 05일 기준  
> ⭐ 시험 빈출 — 파일 위치 암기 필수

---

## 슬래시 커맨드 위치

```
팀 공유 커맨드:  .claude/commands/    ← 버전 관리, 팀 전체
개인 커맨드:     ~/.claude/commands/  ← 개인 전용
```

---

## 커맨드 파일 작성

```markdown
<!-- .claude/commands/review.md -->
코드 리뷰를 수행하세요.

다음 항목을 확인하세요:
1. 보안 취약점 (SQL 인젝션, XSS 등)
2. 성능 문제
3. 코드 스타일 가이드라인 준수
4. 테스트 커버리지

$ARGUMENTS
```

```markdown
<!-- .claude/commands/deploy.md -->
다음 환경에 배포를 준비하세요: $ARGUMENTS

배포 체크리스트:
- [ ] 모든 테스트 통과
- [ ] 변경 이력 업데이트
- [ ] 환경 변수 확인
- [ ] 롤백 계획 수립
```

---

## 커맨드 사용

```bash
# 팀 커맨드 실행
/review src/auth.py

# 인수 없이 실행
/deploy staging
```

---

## $ARGUMENTS 플레이스홀더

```markdown
# argument-hint를 사용한 안내
---
argument-hint: "파일 경로 또는 PR 번호 (선택)"
---

$ARGUMENTS가 있으면 해당 파일/PR을 분석하고,
없으면 최근 변경 파일을 분석하세요.
```

---

## 팀 커맨드 vs 개인 커맨드

```
.claude/commands/ (팀):
- git에 포함
- clone/pull 시 자동 사용 가능
- 팀 표준 워크플로우

~/.claude/commands/ (개인):
- 개인 기기에만
- 개인적 편의 커맨드
- 팀에 영향 없음
```

---

> 🔗 다음: [10.2 Skills 시스템 심화](10_2_skills.md)
