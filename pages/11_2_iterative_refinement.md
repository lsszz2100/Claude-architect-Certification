# 11.2 반복 개선 기법

> 📅 2026년 04월 05일 기준

---

## 반복 개선 패턴

복잡한 작업은 한 번에 완벽하게 하려 하지 말고, 단계적으로 개선합니다.

```
초안 → 검토 → 수정 → 검토 → 최종화
```

---

## 독립 인스턴스 리뷰

```
❌ 자기 리뷰:
Claude A: 코드 생성
Claude A: 자신이 리뷰
→ 같은 편향으로 같은 실수를 놓침

✅ 독립 리뷰:
Claude A: 코드 생성
Claude B: (별도 세션에서) 리뷰
→ 다른 관점으로 실수 발견
```

---

## 멀티패스 접근법

```python
# 멀티패스 코드 리뷰

# 패스 1: 개별 파일 분석
for file in changed_files:
    local_issues = analyze_file(file)

# 패스 2: 크로스파일 통합 분석 (독립 인스턴스)
cross_file_issues = analyze_cross_file_dependencies(
    all_files=changed_files,
    local_results=all_local_issues
)

# 패스 3: 통합 보고서
final_report = integrate_results(local_issues, cross_file_issues)
```

---

## 단계적 구현 전략

```
대규모 기능 추가 시:

1단계: 스켈레톤 구현 (인터페이스만)
    ↓ 검토 후 진행
2단계: 핵심 로직 구현
    ↓ 테스트 작성 및 검토
3단계: 엣지 케이스 처리
    ↓ 통합 테스트
4단계: 성능 최적화
    ↓ 리뷰
완료
```

---

## 피드백 루프

```python
def iterative_improvement(initial_code: str) -> str:
    """반복적으로 코드 개선"""
    
    code = initial_code
    
    for iteration in range(3):
        # 독립 리뷰어 인스턴스
        issues = review_code(code)
        
        if not issues:
            break  # 더 이상 개선 없음
        
        # 발견된 이슈로 개선
        code = fix_issues(code, issues)
    
    return code
```

---

> 🔗 다음: [11.3 CI/CD 파이프라인 통합](11_3_cicd.md)
