# 11.3 CI/CD 파이프라인 통합

> 📅 2026년 04월 05일 기준  
> ⭐ 시험 핵심 — -p 플래그와 --output-format json

---

## CI/CD 핵심 플래그

```bash
# 비대화형 단일 실행
claude -p "코드 분석 수행"

# JSON 출력 (기계 처리용)
claude -p "분석" --output-format json

# 스키마 강제 JSON
claude -p "분석" --output-format json --json-schema '...'
```

---

## 완전한 GitHub Actions 예시

```yaml
name: AI Code Review
on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: AI Review
        run: |
          RESULT=$(claude -p "
          변경된 파일을 검토하세요:
          $(git diff --name-only origin/main...HEAD)
          
          JSON으로 응답하세요.
          " --output-format json)
          
          echo "REVIEW=$RESULT" >> $GITHUB_OUTPUT
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
      
      - name: Fail on Critical Issues
        run: |
          echo "$RESULT" | jq '.issues[] | select(.severity == "critical")' | \
          wc -l | xargs -I{} test {} -eq 0
```

---

## -p 플래그가 없으면?

```
claude "코드 분석"      ← 대화형 모드 시작
                          파이프라인에서 타임아웃!
                          사용자 입력 대기

claude -p "코드 분석"   ← 단일 실행 후 종료
                          파이프라인에 적합
```

---

## 출력 처리

```python
import subprocess
import json

def run_claude_analysis(prompt: str) -> dict:
    result = subprocess.run(
        ["claude", "-p", prompt, "--output-format", "json"],
        capture_output=True, text=True,
        env={"ANTHROPIC_API_KEY": os.environ["ANTHROPIC_API_KEY"]}
    )
    
    return json.loads(result.stdout)

# 사용
analysis = run_claude_analysis("보안 취약점 분석")
critical_issues = [i for i in analysis["issues"] if i["severity"] == "critical"]
```

---

> 🔗 다음: [Chapter 12: 프롬프트 엔지니어링 기초](12_prompt_basics.md)
