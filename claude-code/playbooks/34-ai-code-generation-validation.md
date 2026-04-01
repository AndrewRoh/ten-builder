# AI 코드 생성 → 자동 검증 루프 플레이북

> "코드 생성은 시작일 뿐 — 검증 루프가 코드 품질을 결정한다"

## 개요

AI 코딩 에이전트가 생성한 코드를 자동으로 검증하고, 실패 시 피드백 루프를 돌려 품질을 보장하는 6단계 워크플로우. **생성 1회 → 검증 3회**가 황금 비율이다.

## 왜 검증 루프인가?

AI가 생성한 코드의 첫 번째 버전이 완벽할 확률은 높지 않다. 하지만 "틀린 이유"를 피드백하면 두 번째, 세 번째 시도에서 급격히 좋아진다.

```
1차 생성  → 정확도 ~70%
+ 타입 체크 피드백 → ~85%
+ 테스트 피드백   → ~93%
+ 린트 피드백     → ~97%
```

## 6단계 검증 루프 워크플로우

### Step 1: 스펙 정의 (Spec First)

AI에게 코드를 요청하기 전에 검증 기준을 먼저 정의한다.

```markdown
# 검증 기준 (CLAUDE.md 또는 프롬프트에 포함)

## 타입 안전성
- strict TypeScript (noImplicitAny, strictNullChecks)
- 모든 함수에 명시적 반환 타입

## 테스트 요구사항  
- 최소 커버리지: 80%
- 엣지 케이스: null, undefined, 빈 배열, 경계값
- 에러 시나리오: 네트워크 실패, 타임아웃, 인증 만료

## 린트 규칙
- ESLint + Prettier 통과
- no-any 규칙 적용
- 복잡도 10 이하 (cyclomatic)
```

### Step 2: 코드 생성 + 테스트 동시 요청

```
프롬프트:
"사용자 인증 미들웨어를 구현해줘.
동시에 다음 테스트도 작성해:
1. 유효한 토큰 → 통과
2. 만료된 토큰 → 401
3. 변조된 토큰 → 401
4. 토큰 없음 → 401
5. 리프레시 토큰 시나리오"
```

코드와 테스트를 함께 요청하면 AI가 테스트 가능한 구조로 코드를 설계한다.

### Step 3: 자동 검증 파이프라인 실행

```bash
#!/bin/bash
# validate.sh - AI 생성 코드 자동 검증

set -e

echo "=== Step 1: TypeScript 타입 체크 ==="
npx tsc --noEmit 2>&1 | tee /tmp/type-errors.log
TYPE_EXIT=$?

echo "=== Step 2: ESLint 검사 ==="
npx eslint src/ --format json 2>&1 | tee /tmp/lint-errors.log
LINT_EXIT=$?

echo "=== Step 3: 테스트 실행 ==="
npm test -- --coverage --json 2>&1 | tee /tmp/test-results.log
TEST_EXIT=$?

echo "=== Step 4: 보안 검사 ==="
npx audit-ci --moderate 2>&1 | tee /tmp/security.log
SEC_EXIT=$?

# 결과 요약
echo "---"
echo "타입 체크: $([ $TYPE_EXIT -eq 0 ] && echo '✅' || echo '❌')"
echo "린트: $([ $LINT_EXIT -eq 0 ] && echo '✅' || echo '❌')"
echo "테스트: $([ $TEST_EXIT -eq 0 ] && echo '✅' || echo '❌')"
echo "보안: $([ $SEC_EXIT -eq 0 ] && echo '✅' || echo '❌')"
```

### Step 4: 실패 시 피드백 루프

검증에 실패하면 에러 로그를 AI에게 다시 전달한다.

```
"타입 체크에서 다음 에러가 발생했어:

src/middleware/auth.ts:23:5 - error TS2345: 
  Argument of type 'string | undefined' is not assignable 
  to parameter of type 'string'.

이 에러를 수정하고, 같은 유형의 에러가 다른 곳에도 
있는지 전체 파일을 검토해줘."
```

**핵심**: 에러 메시지만 전달하지 말고, **같은 유형의 에러를 전체적으로 수정**하도록 요청한다.

### Step 5: Claude Code Hooks로 자동화

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "bash -c 'npx tsc --noEmit 2>&1 | head -20'",
        "description": "파일 수정 후 자동 타입 체크"
      }
    ],
    "PreCommit": [
      {
        "command": "bash validate.sh",
        "description": "커밋 전 전체 검증"
      }
    ]
  }
}
```

### Step 6: 품질 게이트 통과 확인

```markdown
## 최종 체크리스트 (PR 생성 전)

### 필수 통과 항목
- [ ] `tsc --noEmit` 에러 0건
- [ ] `eslint` 경고/에러 0건  
- [ ] 테스트 전체 통과 + 커버리지 80%+
- [ ] `npm audit` 취약점 0건 (high/critical)

### 권장 확인 항목
- [ ] 번들 사이즈 증가 < 5KB
- [ ] 응답 시간 회귀 없음
- [ ] 새 의존성 라이선스 확인
```

## 피드백 루프 최적화 패턴

### 패턴 1: 점진적 검증 (Incremental Validation)

전체 검증을 한 번에 돌리지 말고, 단계별로 진행한다.

```
1. 타입 체크 실패 → 수정 → 재검증
2. 타입 통과 → 린트 실패 → 수정 → 재검증  
3. 린트 통과 → 테스트 실패 → 수정 → 재검증
4. 모든 검증 통과 → 커밋
```

이유: 타입 에러가 있으면 린트/테스트도 연쇄 실패하므로, 순서대로 해결하는 게 효율적이다.

### 패턴 2: 검증 결과 컨텍스트 누적

```
"이전 시도에서 다음이 실패했어:
- 1차: 타입 에러 5건 (수정 완료)
- 2차: 테스트 2건 실패 (엣지 케이스 누락)
- 현재 3차: 아래 린트 에러 수정 필요

패턴을 보면 null 체크를 빠뜨리는 경향이 있어. 
이번에는 모든 optional 필드에 대해 
null/undefined 가드를 확인해줘."
```

### 패턴 3: 검증 결과 기반 CLAUDE.md 업데이트

반복되는 실패 패턴을 발견하면 CLAUDE.md에 규칙으로 추가한다.

```markdown
# CLAUDE.md 추가 규칙

## 자주 발생하는 실수 방지
- Optional 체이닝 대신 명시적 null 체크 사용
- async 함수는 반드시 try-catch로 감싸기
- 배열 메서드 체이닝 시 빈 배열 케이스 먼저 처리
```

## 메트릭: 검증 루프 효과 측정

| 메트릭 | 루프 없음 | 1회 루프 | 3회 루프 |
|--------|----------|---------|---------|
| 타입 에러 | 5-10건 | 1-3건 | 0건 |
| 테스트 실패 | 20-30% | 5-10% | 0-2% |
| 린트 경고 | 10-20건 | 2-5건 | 0건 |
| PR 리뷰 코멘트 | 8-15건 | 3-5건 | 0-2건 |
| 수동 수정 시간 | 30-60분 | 10-15분 | 0-5분 |

## 도구별 검증 루프 설정

### Claude Code

```bash
# 자동 검증 모드로 실행
claude "이 코드를 구현하고, 구현 후 npm test와 tsc --noEmit을 
실행해서 에러가 있으면 자동으로 수정해. 
모든 검증을 통과할 때까지 반복해."
```

### Cursor AI

```
Cursor Rules (.cursorrules):
"코드 변경 후 항상 터미널에서 tsc --noEmit과 
npm test를 실행하고, 실패하면 자동 수정하세요."
```

### GitHub Actions CI

```yaml
# .github/workflows/ai-code-validation.yml
name: AI Code Validation
on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npx tsc --noEmit
      - run: npx eslint src/
      - run: npm test -- --coverage
      - name: Coverage Check
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% < 80%"
            exit 1
          fi
```

## 안티패턴: 이렇게 하면 안 된다

### ❌ 에러 메시지만 복사-붙여넣기

```
나쁜 예: "이 에러 수정해: TS2345"
좋은 예: "이 타입 에러를 수정하고, 같은 패턴의 에러가 
        다른 파일에도 있는지 검토해줘."
```

### ❌ 무한 루프 방지 없이 자동 수정

검증 루프는 **최대 3-5회**로 제한한다. 그 이상이면 접근 방식 자체를 재고해야 한다.

### ❌ 모든 검증을 한 번에 피드백

한 번에 20개 에러를 던지면 AI가 혼란스러워한다. **우선순위순으로 5개씩** 피드백하라.

## 정리

검증 루프는 AI 코딩의 **품질 보장 메커니즘**이다:

1. **스펙 먼저** → 검증 기준 정의
2. **코드 + 테스트 동시** → 테스트 가능한 구조 유도
3. **자동 파이프라인** → 수동 확인 제거
4. **피드백 루프** → 점진적 품질 향상
5. **Hooks 자동화** → 실시간 검증
6. **게이트 통과** → 최종 품질 보장

---

*이 플레이북은 텐빌더 채널에서 다루는 AI 코딩 도구 활용 시리즈의 일부입니다.*
*최신 정보는 [@ten-builder](https://youtube.com/@ten-builder) 채널을 구독해주세요.*
