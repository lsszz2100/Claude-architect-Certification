# 08.4 내장 툴 선택 기준

> 📅 2026년 04월 05일 기준  
> ⭐ 시험 자주 출제

---

## 내장 툴 요약

| 툴 | 목적 | 언제 사용 |
|----|------|----------|
| Grep | 파일 내용 검색 | 패턴/코드 찾기 |
| Glob | 파일 경로 찾기 | 파일명 패턴 검색 |
| Read | 파일 읽기 | 전체 파일 내용 |
| Edit | 파일 수정 | 특정 텍스트 교체 |
| Write | 파일 쓰기 | 전체 파일 재작성 |

---

## 상세 사용 기준

### Grep — 내용 검색

```
사용: 파일 내용에서 패턴 찾기
예시:
- "import anthropic" 사용하는 파일 찾기
- TODO 주석 목록 보기
- 특정 함수 호출 찾기
- API 키 하드코딩 검색

❌ 사용 금지: 파일명 찾기 (Glob 사용)
```

### Glob — 경로 검색

```
사용: 파일명 패턴으로 파일 찾기
예시:
- "**/*.test.tsx" — 모든 테스트 파일
- "src/**/*.py" — src 아래 Python 파일
- "*.md" — 현재 디렉토리 마크다운

❌ 사용 금지: 파일 내용 검색 (Grep 사용)
```

### Read vs Edit vs Write

```python
# Read: 파일 내용을 읽을 때
content = Read("/path/to/file.py")

# Edit: 특정 텍스트 교체 (유일한 텍스트여야 함!)
Edit(
    file_path="/path/to/file.py",
    old_string="def old_function():",
    new_string="def new_function():"
)

# Write: 전체 파일 재작성 (반드시 Read 먼저!)
content = Read("/path/to/file.py")  # 먼저 읽기
# ... 내용 수정 ...
Write("/path/to/file.py", modified_content)
```

---

## 결정 플로우차트

```
내용으로 찾고 싶다     → Grep
이름으로 찾고 싶다     → Glob
읽고 싶다              → Read
특정 부분만 바꾸고 싶다 → Edit (유일한 텍스트 필요)
전체를 다시 쓰고 싶다  → Read → Write
새 파일을 만들고 싶다  → Write
```

---

## 시험 함정

```
Q: 특정 함수 정의를 찾으려면?
A: Grep (내용 검색) ← 정답

Q: 모든 설정 파일 목록을 얻으려면?
A: Glob ("**/*.config.js") ← 정답

Q: 큰 파일 일부를 수정하려면?
A: Edit (old_string이 파일 내 유일해야 함) ← 정답
   유일하지 않으면 → Read 후 Write
```

---

> 🔗 다음: [Chapter 09: Claude Code 실전 활용](../09_claude_code.md)
