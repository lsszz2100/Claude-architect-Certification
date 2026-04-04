# 16.3 대규모 코드베이스 탐색

> 📅 2026년 04월 05일 기준

---

## 대규모 코드베이스 탐색 전략

```
목표: 모든 파일을 읽지 않고 필요한 정보를 효율적으로 찾기

전략:
1. 구조 파악 (Glob)
2. 관련 파일 탐색 (Grep)
3. 핵심 파일 읽기 (Read)
4. 수정 (Edit/Write)
```

---

## 단계별 접근

### 1단계: 전체 구조 파악

```python
# Glob으로 파일 구조 파악
python_files = Glob("**/*.py")
test_files = Glob("**/*.test.ts")
config_files = Glob("**/*.config.{js,ts,json}")

# 디렉토리 구조 이해
main_dirs = ["src/", "tests/", "docs/", "scripts/"]
```

### 2단계: 관련 코드 찾기

```python
# Grep으로 관련 코드 검색
auth_usage = Grep("authenticate|login|jwt", type="py")
api_endpoints = Grep("@app.route|@router.get|@router.post", type="py")
TODO_items = Grep("TODO|FIXME|HACK")
```

### 3단계: 핵심 파일 읽기

```python
# 구조 파악 후 핵심 파일만 Read
key_files = [
    "src/auth/authentication.py",
    "src/models/user.py",
    "tests/test_auth.py"
]
```

---

## 멀티에이전트로 대규모 코드베이스 분석

```python
# 코드베이스를 영역으로 분할하여 병렬 분석
coordinator_prompt = """
이 코드베이스를 다음 영역으로 분할하여 분석하세요:

Task 1: src/auth/ 디렉토리 보안 분석
Task 2: src/api/ 디렉토리 API 설계 분석
Task 3: tests/ 디렉토리 테스트 커버리지 분석

세 Task를 동시에 실행하세요.
"""
```

---

## 효율적 탐색 팁

```
✅ 좋은 습관:
- 먼저 Glob/Grep으로 범위 파악
- 필요한 파일만 Read
- 변경 전 반드시 Read 먼저

❌ 나쁜 습관:
- 모든 파일을 순서대로 Read
- Write 전에 Read 안 함
- 관련 없는 파일까지 분석
```

---

> 🔗 다음: [16.4 정보 출처 보존](16_4_provenance.md)
