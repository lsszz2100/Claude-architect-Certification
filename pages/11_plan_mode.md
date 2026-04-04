# Chapter 11: Plan Mode와 개발 워크플로우

> 📅 2026년 04월 05일 기준  
> 🎯 **Domain 3 — 언제 Plan Mode를 사용하는가?**

---

## 11.1 Plan Mode vs Direct Execution

> 🎯 **시험 출제: 복잡도에 따른 선택**

### Plan Mode란?

Plan Mode는 변경을 실행하기 전에 **계획을 먼저 수립**하는 모드입니다.

```bash
# Plan Mode 진입
claude --plan "결제 시스템을 마이크로서비스로 분리해줘"

# Plan 승인 후 실행
# Claude가 계획 제시 → 사용자 검토 → 승인 → 실행
```

### 언제 Plan Mode를 사용하나?

```
Plan Mode 적합:                    Direct Execution 적합:
──────────────────                 ──────────────────────
✅ 대규모 아키텍처 변경             ✅ 단일 파일 버그 수정
✅ 모놀리스 → 마이크로서비스        ✅ 명확한 스택 트레이스로 버그 수정
✅ 45개+ 파일에 영향주는 변경       ✅ 날짜 유효성 검사 추가
✅ 라이브러리 마이그레이션          ✅ 간단한 함수 리팩토링
✅ 여러 유효한 구현 방법 존재       ✅ 타입 힌트 추가
✅ 인프라 요구사항이 다른 접근법    ✅ 문서 업데이트
```

### 의사결정 흐름

```
작업이 다음 조건 중 하나라도 해당하는가?
│
├── 여러 파일에 걸친 대규모 변경?
├── 아키텍처 결정이 필요한가?
├── 여러 유효한 구현 방법이 있는가?
└── 인프라/의존성 변경이 포함되는가?
    │
    YES → Plan Mode 사용
    NO  → Direct Execution
```

---

## 11.2 반복 개선 기법

> 🎯 **시험 출제: 입력/출력 예시가 가장 효과적**

### 기법 1: 구체적 입력/출력 예시

```python
# ❌ 모호한 요청 (불일치 결과 야기)
"날짜 형식을 표준화해줘"

# ✅ 구체적 예시로 변환 요청
"""
다음 날짜 형식을 표준화해주세요:

입력 예시:
- "2024년 1월 15일" → "2024-01-15"
- "Jan 15, 2024" → "2024-01-15"
- "15/01/24" → "2024-01-15"
- "20240115" → "2024-01-15"

출력 형식: YYYY-MM-DD (ISO 8601)
처리 불가 시: null 반환 (예외 발생 금지)
"""
```

### 기법 2: 테스트 주도 반복

```python
# Step 1: 먼저 테스트 작성
"""
다음 테스트를 통과하는 함수를 작성해줘:

def test_extract_invoice_data():
    # 기본 케이스
    result = extract_invoice_data(invoice_text)
    assert result["invoice_number"] == "INV-2024-001"
    assert result["total_amount"] == 1500.00
    assert result["due_date"] == "2024-02-15"
    
    # 엣지 케이스: 날짜 없음
    result = extract_invoice_data(no_date_invoice)
    assert result["due_date"] is None  # 예외 아님!
    
    # 엣지 케이스: 부가세 포함 금액
    result = extract_invoice_data(vat_invoice)
    assert result["subtotal"] == 1000.00
    assert result["vat"] == 100.00
    assert result["total_amount"] == 1100.00
"""

# Step 2: 실패한 테스트 공유하며 반복 개선
"이 테스트가 실패합니다: [실패 메시지]. 수정해줘."
```

### 기법 3: 인터뷰 패턴

```python
# 모르는 도메인에서 구현 전 질문 유도
"""
레거시 결제 시스템에 캐시를 추가하려고 합니다.
구현하기 전에 고려해야 할 사항들을 질문해주세요.
"""

# Claude가 질문:
# - 캐시 무효화 전략은 어떻게 할 계획인가요?
# - 결제 실패 시 캐시된 데이터는 어떻게 처리하나요?
# - 분산 환경에서 캐시 일관성은 어떻게 보장하나요?
# → 개발자가 미처 생각하지 못한 고려사항 발굴!
```

### 기법 4: 상호작용 문제 vs 독립 문제

```python
# ❌ 잘못된 방법: 상호작용하는 문제를 하나씩 순차 수정
fix_bug_1()  # 버그 1 수정
fix_bug_2()  # 버그 2 수정 → 버그 1 수정 영향받을 수 있음!

# ✅ 올바른 방법: 상호작용하는 문제는 한 번에 전달
"""
다음 두 문제가 서로 영향을 줍니다:
1. 환불 계산 오류 (processor.py:145)
2. 세금 계산 오류 (tax.py:87)

이 두 문제는 tax_amount 변수를 공유하므로 함께 수정해야 합니다.
"""
```

---

## 11.3 CI/CD 파이프라인 통합

> 🎯 **시험 출제: -p 플래그, --output-format json**

### 비대화형 모드 실행

```bash
# ✅ CI/CD에서 올바른 실행 방법
claude -p "이 PR의 보안 취약점을 분석해주세요"
# 또는
claude --print "이 PR의 보안 취약점을 분석해주세요"

# ❌ 잘못된 방법 (CI에서 무한 대기 발생)
claude "이 PR의 보안 취약점을 분석해주세요"
# → 대화형 입력 대기로 파이프라인 중단!
```

### JSON 구조화 출력

```bash
# 기계가 읽을 수 있는 JSON으로 출력
claude -p "코드를 검토하고 이슈를 보고해주세요" \
    --output-format json \
    --json-schema ./schemas/review-result.json

# 결과 예시:
{
  "issues": [
    {
      "file": "payment/processor.py",
      "line": 145,
      "severity": "critical",
      "category": "security",
      "message": "SQL 인젝션 취약점",
      "suggestion": "파라미터화된 쿼리 사용"
    }
  ],
  "summary": "3개의 Critical, 2개의 Warning 발견"
}
```

### GitHub Actions 통합 예시

```yaml
# .github/workflows/ai-code-review.yml
name: AI Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  claude-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code
      
      - name: Run AI Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude -p "변경된 파일을 검토하고 이슈를 보고해주세요" \
            --output-format json \
            --json-schema ./schemas/review.json \
            > review-results.json
      
      - name: Post Review Comments
        uses: actions/github-script@v7
        with:
          script: |
            const results = require('./review-results.json');
            for (const issue of results.issues) {
              await github.rest.pulls.createReviewComment({
                ...context.repo,
                pull_number: context.issue.number,
                body: `**${issue.severity}**: ${issue.message}\n\n💡 ${issue.suggestion}`,
                path: issue.file,
                line: issue.line
              });
            }
```

### 세션 격리의 중요성

```python
# ❌ 잘못된 방법: 코드 생성과 리뷰를 같은 세션에서
generate_code()  # 코드 생성
review_code()    # 같은 세션에서 리뷰 → 자기 코드 리뷰라 편향!

# ✅ 올바른 방법: 독립적인 리뷰 인스턴스
generate_code()  # 세션 A에서 생성
# 새 Claude 인스턴스로 리뷰 (이전 맥락 없음)
review_code_independently()  # 세션 B에서 리뷰 → 더 객관적!
```

---

## 📝 챕터 요약

- Plan Mode: 대규모, 다중 파일, 아키텍처 결정 작업에 사용
- Direct Execution: 단일 파일, 명확한 버그 수정에 사용
- 가장 효과적인 개선 기법: 구체적인 입력/출력 예시 제공
- CI/CD: `-p` (비대화형), `--output-format json` (기계 처리 가능)
- 코드 생성과 리뷰는 별도 세션으로 격리 → 더 객관적인 리뷰

---

> 🔗 다음 챕터: [프롬프트 엔지니어링 기초](12_prompt_basics.md)
