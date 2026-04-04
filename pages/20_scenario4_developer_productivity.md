# Chapter 20: 시나리오 4 — 개발자 생산성 도구

> 📅 2026년 04월 05일 기준  
> 🎯 실제 시험 시나리오 4 해설

---

## 시나리오 개요

> 당신은 내부 개발자 도구 팀에 있습니다.
> Claude를 사용하여 개발자의 일상적인 작업(코드 리뷰, 버그 분석, 문서화)을 자동화합니다.
> 여러 개발자 도구와 통합된 MCP 서버 기반 시스템을 설계합니다.

---

## 핵심 아키텍처: 툴 배분 전략

### 문제: 너무 많은 툴

```
❌ 잘못된 설계: 모든 기능을 하나의 에이전트에
tools = [
    "search_code", "read_file", "write_file", "run_tests",
    "check_ci", "post_comment", "assign_reviewer", "create_pr",
    "update_jira", "send_slack", "check_coverage", "lint_code",
    "analyze_performance", "check_security", "update_docs",
    "create_ticket", "search_docs", "run_migration"
]
# 18개 툴 → 선택 신뢰도 저하!
```

```
✅ 올바른 설계: 전문화된 에이전트로 분리
코드 리뷰 에이전트: 4-5개 툴
  - read_file, search_code, lint_code, post_comment, check_coverage

CI/CD 에이전트: 4-5개 툴
  - check_ci, run_tests, get_build_logs, trigger_deploy, notify_team

문서화 에이전트: 3-4개 툴
  - read_file, update_docs, search_docs, create_ticket
```

### 툴 수와 선택 신뢰도

```
툴 수    선택 신뢰도
 4개    ██████████ 매우 높음
 8개    ████████   높음  
12개    ██████     보통
18개    ████       낮음 ← 18개가 기준점
20개+   ██         매우 낮음
```

---

## MCP 서버 설계

### 코드 리뷰 MCP 서버

```python
# code_review_mcp_server.py
from mcp import MCPServer, tool

server = MCPServer("code-review-tools")

@tool(
    name="analyze_code_quality",
    description="""코드 품질을 분석합니다.
    
    사용 시점:
    - PR 리뷰 시 코드 품질 자동 검사
    - 특정 파일의 복잡도, 중복, 코딩 컨벤션 확인
    
    입력:
    - file_path: 분석할 파일 경로
    - checks: 수행할 검사 목록 (complexity, duplication, style)
    
    반환:
    - issues: 발견된 문제 목록 (심각도 포함)
    - score: 품질 점수 (0-100)
    
    주의: security 검사는 scan_security_vulnerabilities 사용"""
)
async def analyze_code_quality(file_path: str, checks: list[str]):
    """코드 품질 분석 구현"""
    pass


@tool(
    name="scan_security_vulnerabilities",
    description="""보안 취약점을 스캔합니다.
    
    사용 시점: analyze_code_quality와 구별하여 보안 전용 검사
    
    입력:
    - file_path: 분석할 파일 경로
    - scan_depth: 'quick' | 'thorough'
    
    반환:
    - vulnerabilities: OWASP 기반 취약점 목록
    - severity: critical/high/medium/low
    - fix_suggestions: 수정 제안"""
)
async def scan_security_vulnerabilities(file_path: str, scan_depth: str = "quick"):
    """보안 취약점 스캔 구현"""
    pass
```

### .mcp.json 설정

```json
{
  "mcpServers": {
    "code-review": {
      "type": "stdio",
      "command": "python",
      "args": ["-m", "code_review_mcp_server"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}",
        "JIRA_API_KEY": "${JIRA_API_KEY}",
        "LOG_LEVEL": "info"
      }
    },
    "cicd-tools": {
      "type": "stdio",
      "command": "node",
      "args": ["cicd_mcp_server.js"],
      "env": {
        "JENKINS_URL": "${JENKINS_URL}",
        "JENKINS_TOKEN": "${JENKINS_TOKEN}"
      }
    }
  }
}
```

---

## 내장 툴 최적 활용

### 툴 선택 기준

```
시나리오                          최적 툴
───────────────────────────────────────────
특정 함수 정의 찾기              → Grep
모든 테스트 파일 목록            → Glob (*.test.tsx)
파일 전체 내용 읽기              → Read
특정 텍스트 교체 (유일한 텍스트) → Edit
파일 전체 재작성                 → Write (Read 먼저!)
```

### 올바른 툴 사용 예시

```python
# 코드 리뷰 에이전트의 작업 흐름

# 1. 변경된 파일 목록 찾기
# Glob: "**/*.py" 패턴으로 Python 파일 검색

# 2. 특정 패턴 검색 (예: TODO 주석)
# Grep: "TODO|FIXME|HACK" 패턴으로 내용 검색

# 3. 특정 파일 내용 읽기
# Read: 전체 파일 내용 확인

# 4. 코드 수정 (특정 함수 교체)
# Edit: 고유한 함수명으로 정확한 위치 교체

# 5. 새 파일 생성 (테스트 파일)
# Write: 새 파일 작성
```

---

## Hooks를 활용한 자동화

### PostToolUse 훅 설계

```python
# .claude/hooks/post_tool_use.py
import json
import sys

def normalize_code_output(tool_name: str, tool_result: dict) -> dict:
    """툴 결과를 정규화하는 PostToolUse 훅"""
    
    if tool_name == "run_tests":
        # 테스트 결과 표준화
        return {
            "passed": tool_result.get("exit_code") == 0,
            "total": tool_result.get("test_count", 0),
            "failed": tool_result.get("failures", []),
            "coverage": tool_result.get("coverage_percent", None)
        }
    
    elif tool_name == "check_ci":
        # CI 상태 정규화
        raw_status = tool_result.get("status", "unknown")
        status_map = {
            "SUCCESS": "passed",
            "FAILURE": "failed",
            "RUNNING": "in_progress",
            "PENDING": "queued"
        }
        return {
            "status": status_map.get(raw_status, raw_status),
            "duration_minutes": tool_result.get("duration", 0) / 60,
            "url": tool_result.get("build_url")
        }
    
    return tool_result


# stdin으로 툴 결과 받기
tool_event = json.loads(sys.stdin.read())
normalized = normalize_code_output(
    tool_event["tool_name"],
    tool_event["result"]
)
print(json.dumps(normalized))
```

### PreToolUse 훅 — 정책 강제

```python
# .claude/hooks/pre_tool_use.py
def enforce_policy(tool_name: str, tool_input: dict) -> bool:
    """툴 사용 전 정책 검사"""
    
    # 프로덕션 환경에 직접 배포 차단
    if tool_name == "deploy":
        if tool_input.get("environment") == "production":
            if not tool_input.get("approval_ticket"):
                print(json.dumps({
                    "blocked": True,
                    "reason": "프로덕션 배포는 승인 티켓이 필요합니다"
                }))
                return False
    
    # 메인 브랜치 직접 커밋 차단
    if tool_name == "git_commit":
        if tool_input.get("branch") in ["main", "master"]:
            print(json.dumps({
                "blocked": True,
                "reason": "메인 브랜치에 직접 커밋할 수 없습니다"
            }))
            return False
    
    return True
```

---

## 에러 처리 전략

### 구조화된 에러 응답

```python
# 내부 도구 에러 응답 설계
def get_error_response(error_type: str, context: dict) -> dict:
    
    if error_type == "test_failure":
        return {
            "isError": True,
            "errorCategory": "business",    # 비즈니스 규칙 위반 (재시도 불가)
            "isRetryable": False,
            "message": "테스트 실패: 코드를 수정 후 다시 실행하세요",
            "details": context.get("failed_tests", [])
        }
    
    elif error_type == "timeout":
        return {
            "isError": True,
            "errorCategory": "transient",   # 일시적 오류 (재시도 가능)
            "isRetryable": True,
            "message": "CI 서버 응답 시간 초과",
            "retry_after_seconds": 30
        }
    
    elif error_type == "permission":
        return {
            "isError": True,
            "errorCategory": "permission",  # 권한 오류 (재시도 불가)
            "isRetryable": False,
            "message": "해당 저장소에 접근 권한이 없습니다"
        }
```

---

## 시나리오 기반 예상 문제

### Q: 툴 배분 결정

상황: 코드 리뷰 에이전트에 18개의 툴을 제공했더니 에이전트가 자주 잘못된 툴을 선택합니다.

가장 효과적인 해결책은?

A) 각 툴의 설명을 더 자세하게 작성  
B) Few-shot 예시를 추가하여 올바른 툴 선택 패턴 교육  
C) 에이전트를 기능별로 분리하여 각 에이전트가 4-5개 툴만 사용하도록 설계  
D) 더 강력한 모델(Opus)으로 교체  

정답: C — 너무 많은 툴(18개)은 선택 신뢰도 저하. 전문화된 에이전트로 분리하여 툴 수 줄이기

---

### Q: Grep vs Glob 선택

상황: 특정 import 문(`import anthropic`)을 사용하는 모든 Python 파일을 찾고 싶습니다.

올바른 툴은?

A) Glob ("*.py")  
B) Grep ("import anthropic", glob: "*.py")  
C) Read (모든 파일 읽기)  
D) Bash (find + grep)  

정답: B — 파일 내용에서 패턴 검색 = Grep. Glob은 파일 경로 패턴으로 찾을 때 사용

---

## 📝 챕터 요약

| 개념 | 핵심 내용 |
|------|---------|
| 툴 수 제한 | 18개 이하 (4-5개가 이상적) |
| 에이전트 전문화 | 기능별 분리로 선택 신뢰도 향상 |
| MCP 서버 | .mcp.json으로 팀 공유 |
| Hooks | PostToolUse (정규화), PreToolUse (차단) |
| 내장 툴 | Grep(내용), Glob(경로), Read, Edit, Write |
| 에러 분류 | transient(재시도 가능) vs others(불가) |

---

> 🔗 다음 챕터: [시나리오 5 — CI/CD 통합](21_scenario5_cicd.md)
