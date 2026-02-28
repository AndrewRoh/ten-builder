# PR 리뷰 워크플로

> AI를 활용한 체계적인 Pull Request 리뷰 프로세스

## PR 생성 전 (작성자)

### 셀프 리뷰

```bash
# 변경사항 전체 리뷰
claude "현재 브랜치의 변경사항을 리뷰해줘.
체크리스트:
- [ ] 보안 취약점
- [ ] 에러 핸들링 누락
- [ ] 타입 안전성
- [ ] 성능 이슈
- [ ] 테스트 커버리지"
```

### PR 설명 자동 생성

```bash
# PR 본문 초안 생성
claude "main 대비 변경사항을 분석하고 PR 설명을 작성해줘.
포맷:
## 변경 내용
## 왜 이렇게 했나
## 테스트 방법
## 스크린샷 (해당 시)"
```

## PR 리뷰 시 (리뷰어)

### 1차: 전체 파악

```bash
# PR의 변경사항 요약
gh pr diff [PR_NUMBER] | claude "이 PR의 변경사항을 요약해줘"
```

### 2차: 심층 리뷰

```bash
# 관점별 리뷰
gh pr diff [PR_NUMBER] | claude "다음 관점에서 리뷰해줘:
1. 이 변경이 기존 기능을 깨뜨리지 않는지
2. 엣지 케이스 처리
3. 성능 영향"
```

### 3차: 리뷰 코멘트 작성

```bash
claude "리뷰 결과를 GitHub PR 코멘트 형식으로 작성해줘.
- 반드시 고쳐야 할 것 (🔴)
- 권장 사항 (🟡)
- 칭찬할 점 (🟢)"
```

## 자동화 (선택)

### GitHub Actions 연동

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review
on: [pull_request]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: AI Review
        # Claude Code를 CI에서 실행하는 방법은
        # 공식 문서 참조: https://docs.anthropic.com/claude-code
```

## 리뷰 문화 팁

- AI 리뷰는 **1차 필터** — 최종 판단은 사람이
- "AI가 OK했으니 머지" ❌ → "AI가 놓친 것은?" ✅
- AI 리뷰 결과를 팀과 공유 → 리뷰 기준 통일 효과
