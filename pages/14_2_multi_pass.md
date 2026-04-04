# 14.2 멀티패스 리뷰 설계

> 📅 2026년 04월 05일 기준  
> ⭐ **시험 핵심 — 자기 리뷰 vs 독립 인스턴스**

---

## 자기 리뷰의 한계

```
❌ 자기 리뷰:
Claude A: "이 코드를 작성했습니다"
Claude A: "이제 내 코드를 리뷰하겠습니다"
→ 같은 모델 인스턴스, 같은 편향
→ 자신의 실수를 놓침

✅ 독립 인스턴스 리뷰:
Claude A: 코드 작성
Claude B: (새 세션, 다른 관점) 리뷰
→ 다른 관점으로 실수 발견
```

---

## 멀티패스 리뷰 구조

```python
def multipass_review(files: list[str]) -> dict:
    """
    1패스: 개별 파일 분석 (로컬 이슈)
    2패스: 크로스파일 통합 분석 (전역 이슈)
    """
    
    # 패스 1: 각 파일 개별 분석
    local_results = {}
    for file_path in files:
        content = read_file(file_path)
        local_results[file_path] = analyze_single_file(content)
    
    # 패스 2: 크로스파일 통합 (독립 인스턴스!)
    cross_file_issues = analyze_cross_dependencies(
        files=files,
        local_results=local_results
    )
    
    return {
        "local_issues": local_results,
        "cross_file_issues": cross_file_issues
    }
```

---

## 독립 인스턴스 구현

```python
def independent_review(code: str) -> str:
    """완전히 새로운 Claude 인스턴스로 리뷰"""
    
    # 새 클라이언트 = 새 인스턴스 (이전 컨텍스트 없음)
    reviewer = anthropic.Anthropic()
    
    response = reviewer.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2048,
        messages=[{
            "role": "user",
            "content": f"""다음 코드를 독립적으로 리뷰하세요.
이 코드가 어떻게 작성되었는지 모르는 상태로 분석하세요:

{code}"""
        }]
    )
    
    return response.content[0].text
```

---

## 시험 핵심 정리

```
자기 리뷰 < 독립 인스턴스 리뷰

멀티패스 순서:
1. 파일별 로컬 분석
2. 크로스파일 통합 분석

독립 인스턴스 = 새 세션, 새 컨텍스트
```

---

> 🔗 다음: [Chapter 15: 컨텍스트 관리 전략](../15_context_management.md)
