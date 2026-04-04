# Chapter 21: 시나리오 5 — CI/CD 통합

> 📅 2026년 04월 05일 기준  
> 🎯 실제 시험 시나리오 5 해설

---

## 시나리오 개요

> 당신은 DevOps 팀에서 Claude를 CI/CD 파이프라인에 통합하고 있습니다.
> 자동화된 코드 품질 검사, 보안 스캔, 테스트 분석을 구현합니다.
> 파이프라인은 완전 비대화형(headless)으로 실행되어야 합니다.

---

## 핵심 개념: 비대화형 모드

### -p (--print) 플래그

```bash
# CI/CD 환경에서 Claude Code 사용
# -p 플래그: 비대화형 단일 실행 모드

# 기본 사용법
claude -p "PR의 코드를 검토하고 문제점을 나열하세요"

# JSON 출력 (기계 처리 가능)
claude -p "보안 취약점을 분석하세요" --output-format json

# 스키마 강제 JSON 출력
claude -p "코드 분석" \
  --output-format json \
  --json-schema '{"type": "object", "properties": {"issues": {"type": "array"}, "score": {"type": "number"}}}'

# 파일 입력
claude -p "$(cat pr_diff.txt)를 분석하세요" --output-format json
```

### -p 플래그의 의미

```
일반 모드:            비대화형 모드 (-p):
┌─────────┐           ┌─────────┐
│ 사용자   │ ←대화→   │ 자동화   │
│ Claude  │           │ Claude  │
└─────────┘           └─────────┘
                      - 단일 요청/응답
                      - stdin/stdout
                      - CI/CD 파이프라인에 임베드
                      - 자동 종료
```

---

## GitHub Actions 통합

### 완전한 파이프라인 예시

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review Pipeline

on:
  pull_request:
    branches: [main, develop]
    types: [opened, synchronize]

jobs:
  security-scan:
    name: Security Analysis
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 전체 히스토리
      
      - name: Setup Claude Code
        run: npm install -g @anthropic-ai/claude-code
      
      - name: Get Changed Files
        id: changed
        run: |
          FILES=$(git diff --name-only origin/main...HEAD | tr '\n' ' ')
          echo "files=$FILES" >> $GITHUB_OUTPUT
      
      - name: Run Security Analysis
        id: security
        run: |
          RESULT=$(claude -p "
          다음 파일들의 보안 취약점을 분석하세요: ${{ steps.changed.outputs.files }}
          
          각 취약점에 대해 다음을 포함하세요:
          - 파일명과 라인 번호
          - 취약점 유형 (OWASP 기반)
          - 심각도 (critical/high/medium/low)
          - 수정 방법
          " --output-format json)
          echo "result=$RESULT" >> $GITHUB_OUTPUT
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
      
      - name: Post Review Comment
        uses: actions/github-script@v7
        with:
          script: |
            const result = JSON.parse('${{ steps.security.outputs.result }}');
            const issues = result.issues || [];
            
            if (issues.length > 0) {
              const comment = issues.map(i => 
                `**${i.severity.toUpperCase()}**: ${i.file}:${i.line}\n${i.description}`
              ).join('\n\n');
              
              await github.rest.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.issue.number,
                body: `## 🔐 보안 분석 결과\n\n${comment}`
              });
            }
      
      - name: Fail on Critical Issues
        run: |
          RESULT='${{ steps.security.outputs.result }}'
          CRITICAL=$(echo $RESULT | jq '[.issues[] | select(.severity == "critical")] | length')
          
          if [ "$CRITICAL" -gt "0" ]; then
            echo "❌ 심각한 보안 취약점 발견: ${CRITICAL}개"
            exit 1
          fi
```

### 테스트 분석 파이프라인

```yaml
  test-analysis:
    name: Test Analysis
    runs-on: ubuntu-latest
    needs: security-scan
    steps:
      - name: Run Tests
        id: tests
        run: |
          pytest --json-report --json-report-file=test-results.json || true
      
      - name: Analyze Test Failures
        if: always()
        run: |
          if [ -f test-results.json ]; then
            FAILURES=$(cat test-results.json | jq '.failures')
            
            if [ "$FAILURES" != "[]" ]; then
              claude -p "
              다음 테스트 실패를 분석하고 수정 방법을 제안하세요:
              
              $(cat test-results.json | jq '.failures')
              
              각 실패에 대해:
              1. 실패 원인
              2. 관련 코드 파일
              3. 구체적인 수정 방법
              " --output-format json > failure-analysis.json
            fi
          fi
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

---

## Batch API 활용

### 대량 코드 분석

```python
import anthropic
import json

client = anthropic.Anthropic()

def batch_analyze_files(file_paths: list[str]) -> str:
    """Batch API로 여러 파일을 비용 효율적으로 분석"""
    
    # Batch API: 50% 비용 절감, 최대 24시간 처리
    # 적합: 비실시간, 비차단 작업
    # 부적합: pre-merge 체크 (즉각 응답 필요)
    
    requests = []
    for i, file_path in enumerate(file_paths):
        with open(file_path, 'r') as f:
            content = f.read()
        
        requests.append({
            "custom_id": f"analysis-{i}",
            "params": {
                "model": "claude-sonnet-4-6",
                "max_tokens": 1024,
                "messages": [{
                    "role": "user",
                    "content": f"""파일 {file_path}의 코드 품질을 분석하세요:

{content}

JSON 형식으로 응답:
{{
  "complexity": "low|medium|high",
  "issues": [...],
  "improvements": [...]
}}"""
                }]
            }
        })
    
    # 배치 생성
    batch = client.messages.batches.create(requests=requests)
    print(f"배치 ID: {batch.id}, 상태: {batch.processing_status}")
    
    return batch.id


def get_batch_results(batch_id: str) -> dict:
    """배치 결과 수집"""
    
    results = {}
    for result in client.messages.batches.results(batch_id):
        if result.result.type == "succeeded":
            custom_id = result.custom_id
            content = result.result.message.content[0].text
            results[custom_id] = json.loads(content)
    
    return results


# 야간 코드베이스 분석 (비차단 작업에 적합)
def nightly_analysis():
    import glob
    
    python_files = glob.glob("**/*.py", recursive=True)
    batch_id = batch_analyze_files(python_files)
    
    # 24시간 내 처리 완료, 실시간 대기 불필요
    print(f"야간 분석 시작: {batch_id}")
    print("내일 아침 결과를 확인하세요")
```

---

## 출력 형식 처리

### JSON 출력 파싱

```python
import subprocess
import json

def run_claude_analysis(prompt: str, schema: dict = None) -> dict:
    """Claude Code를 파이프라인에서 실행"""
    
    cmd = ["claude", "-p", prompt, "--output-format", "json"]
    
    if schema:
        cmd.extend(["--json-schema", json.dumps(schema)])
    
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True,
        env={"ANTHROPIC_API_KEY": os.environ["ANTHROPIC_API_KEY"]}
    )
    
    if result.returncode != 0:
        raise RuntimeError(f"Claude Code 실행 실패: {result.stderr}")
    
    return json.loads(result.stdout)


# 파이프라인에서 사용
analysis_schema = {
    "type": "object",
    "properties": {
        "score": {"type": "number", "minimum": 0, "maximum": 100},
        "issues": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "severity": {"type": "string", "enum": ["critical", "high", "medium", "low"]},
                    "message": {"type": "string"},
                    "file": {"type": "string"},
                    "line": {"type": "integer"}
                }
            }
        },
        "passed": {"type": "boolean"}
    },
    "required": ["score", "issues", "passed"]
}

result = run_claude_analysis(
    "변경된 코드의 품질을 분석하세요",
    schema=analysis_schema
)

print(f"품질 점수: {result['score']}/100")
print(f"합격: {result['passed']}")
```

---

## 시나리오 기반 예상 문제

### Q: CI/CD 비대화형 모드

상황: GitHub Actions에서 Claude Code를 실행하려고 합니다. 파이프라인이 Claude Code의 응답을 기다리다 타임아웃됩니다.

가장 먼저 확인해야 할 것은?

A) ANTHROPIC_API_KEY 환경 변수 설정 여부  
B) -p (--print) 플래그 사용 여부 — 비대화형 모드  
C) 네트워크 연결 문제  
D) 모델 이름 정확성  

정답: B — CI/CD 파이프라인에는 반드시 -p 플래그 사용 (비대화형 모드)

---

### Q: Batch API vs 실시간 처리

상황: 다음 두 작업 중 Batch API에 적합한 것은?

A) PR 머지 전 자동 코드 리뷰 (결과 즉시 필요)  
B) 매주 일요일 밤 전체 코드베이스 품질 분석 (다음 날 아침 확인)  

정답: B — Batch API는 비차단 작업(야간 분석, 주간 보고서)에 적합. pre-merge 체크는 즉각 응답이 필요하므로 부적합

---

## 📝 챕터 요약

| 개념 | 핵심 내용 |
|------|---------|
| -p 플래그 | CI/CD 비대화형 모드 필수 |
| --output-format json | 기계 처리 가능 출력 |
| --json-schema | 스키마 강제 적용 |
| Batch API | 50% 비용 절감, 24시간, 비차단 작업 |
| pre-merge 체크 | 실시간 필요 → Batch API 부적합 |

---

> 🔗 다음 챕터: [시나리오 6 — 데이터 추출 파이프라인](22_scenario6_data_extraction.md)
