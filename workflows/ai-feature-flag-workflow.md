# AI 코딩 에이전트 + Feature Flag 안전 배포 워크플로우

> AI가 생성한 코드를 Feature Flag로 안전하게 배포하고, 문제 시 즉시 롤백하는 워크플로우

## 왜 AI 코드에 Feature Flag가 필요한가?

AI가 생성한 코드는 로컬 테스트를 통과해도 프로덕션에서 예상치 못한 동작을 할 수 있다. Feature Flag를 사용하면:

- **점진적 롤아웃**: 전체 사용자가 아닌 1% → 10% → 50% → 100%
- **즉시 롤백**: 코드 배포 없이 플래그만 끄면 원복
- **A/B 테스트**: AI 생성 코드 vs 기존 코드 성능 비교

## 전체 워크플로우

```
AI 코드 생성 → Feature Flag 래핑 → PR 생성 → 
CI 검증 → 머지 → 1% 롤아웃 → 모니터링 → 
점진적 확대 → 100% → Flag 정리
```

## Step 1: AI에게 Feature Flag 포함 코드 요청

### CLAUDE.md 규칙 추가

```markdown
# Feature Flag 규칙

## 신규 기능 구현 시
- 모든 새로운 기능은 Feature Flag로 감싸서 구현
- Flag 이름 규칙: `feature_<기능명>_<날짜>`
- Flag OFF 시 기존 동작 보장 (fallback 필수)

## Flag 구현 패턴
```typescript
if (featureFlags.isEnabled('feature_new_auth_20260402')) {
  // 새 구현
} else {
  // 기존 구현 (fallback)
}
```
```

### AI 프롬프트 예시

```
"결제 시스템의 재시도 로직을 exponential backoff로 
개선해줘. Feature Flag로 감싸서 구현하고,
플래그 OFF 시 기존 고정 3회 재시도 로직이 작동하도록 해."
```

## Step 2: Feature Flag 라이브러리 설정

### TypeScript + LaunchDarkly 예시

```typescript
// src/config/feature-flags.ts
import { LDClient, init } from 'launchdarkly-node-server-sdk';

let client: LDClient;

export async function initFeatureFlags() {
  client = init(process.env.LAUNCHDARKLY_SDK_KEY!);
  await client.waitForInitialization();
}

export async function isEnabled(
  flagKey: string, 
  userId?: string,
  defaultValue: boolean = false
): Promise<boolean> {
  const user = userId 
    ? { key: userId } 
    : { key: 'anonymous', anonymous: true };
  
  return client.variation(flagKey, user, defaultValue);
}
```

### 경량 대안: 환경변수 기반 (소규모 팀)

```typescript
// src/config/simple-flags.ts
const FLAGS: Record<string, boolean> = {
  feature_new_auth_20260402: 
    process.env.FLAG_NEW_AUTH === 'true',
  feature_payment_retry_20260402: 
    process.env.FLAG_PAYMENT_RETRY === 'true',
};

export function isEnabled(flagKey: string): boolean {
  return FLAGS[flagKey] ?? false;
}
```

## Step 3: AI 생성 코드에 Flag 적용 패턴

### 패턴 A: 함수 단위 교체

```typescript
// 기존 코드 유지 + 새 코드 공존
async function processPayment(order: Order): Promise<Result> {
  if (await isEnabled('feature_payment_v2_20260402', order.userId)) {
    return processPaymentV2(order);  // AI가 생성한 새 버전
  }
  return processPaymentV1(order);    // 기존 검증된 버전
}
```

### 패턴 B: 미들웨어 단위 교체

```typescript
// Express 미들웨어에서 분기
app.use(async (req, res, next) => {
  if (await isEnabled('feature_new_rate_limiter')) {
    return newRateLimiter(req, res, next);
  }
  return legacyRateLimiter(req, res, next);
});
```

### 패턴 C: React 컴포넌트 교체

```tsx
function CheckoutPage() {
  const isNewCheckout = useFeatureFlag('feature_checkout_v2');
  
  return isNewCheckout 
    ? <NewCheckoutFlow />   // AI 생성 새 UI
    : <LegacyCheckout />;   // 기존 UI
}
```

## Step 4: 점진적 롤아웃 전략

### 롤아웃 스케줄

```
Day 0: 내부 팀 (개발자 + QA)
Day 1: 1% 외부 사용자 (카나리아)
Day 2: 모니터링 확인 후 10%
Day 4: 25% (주말 전 중단)
Day 7: 50%
Day 10: 75%
Day 14: 100%
```

### 자동 롤백 트리거

```yaml
# 다음 조건 중 하나라도 충족 시 자동 OFF
rollback_triggers:
  error_rate: "> 0.5% (기준선 대비 2배)"
  latency_p99: "> 500ms (기준선 대비 3배)"
  crash_rate: "> 0.01%"
  revenue_impact: "> -2%"
```

## Step 5: 모니터링 대시보드

### 필수 메트릭

```
1. Error Rate (flag ON vs OFF 그룹 비교)
2. Latency P50/P95/P99
3. 비즈니스 메트릭 (전환율, 결제 성공률)
4. 사용자 행동 변화 (이탈률, 체류 시간)
```

### 알림 설정 예시

```yaml
# datadog-monitors.yml
monitors:
  - name: "AI Feature - Error Rate Spike"
    query: |
      avg(last_5m):sum:errors{feature_flag:new_auth} / 
      sum:requests{feature_flag:new_auth} > 0.005
    message: "Feature flag 'new_auth' error rate > 0.5%"
    escalation: "자동 롤백 트리거됨"
```

## Step 6: Flag 정리 (Technical Debt 방지)

AI 코드가 검증되면 Feature Flag를 정리한다.

### 정리 체크리스트

```markdown
## Flag 정리 PR 체크리스트
- [ ] 100% 롤아웃 후 최소 2주 안정 운영
- [ ] 기존 코드 (fallback) 제거
- [ ] Flag 분기 조건 제거
- [ ] 관련 테스트 업데이트
- [ ] Flag 서비스에서 삭제
- [ ] 문서 업데이트
```

### AI에게 Flag 정리 요청

```
"feature_payment_v2_20260402 플래그를 정리해줘.
V2 코드만 남기고, V1 코드와 플래그 분기를 제거해.
관련 테스트도 업데이트해."
```

## CI/CD 파이프라인 통합

### GitHub Actions 예시

```yaml
# .github/workflows/feature-flag-deploy.yml
name: Feature Flag Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
      
      - name: Deploy to Staging
        run: npm run deploy:staging
        
      - name: Enable Flag (1% Canary)
        run: |
          curl -X PATCH "$LD_API/flags/$PROJECT_KEY/$FLAG_KEY" \
            -H "Authorization: $LD_API_KEY" \
            -d '{"environments":{"production":{"on":true,"fallthrough":{"rollout":{"variations":[{"variation":0,"weight":1000},{"variation":1,"weight":99000}]}}}}}'
      
      - name: Health Check (5min)
        run: sleep 300 && bash check-health.sh
        
      - name: Increase to 10% (if healthy)
        if: success()
        run: bash rollout.sh 10
```

## 안티패턴

### ❌ 플래그 없이 AI 코드 직접 배포

리스크가 너무 높다. 롤백 시 재배포 필요.

### ❌ 플래그를 영원히 남겨두기

2주 이상 100% ON인 플래그는 정리해야 한다. 기술 부채가 된다.

### ❌ 너무 세밀한 플래그

한 줄짜리 변경에 플래그를 거는 건 오버엔지니어링. **의미 있는 기능 단위**로 플래그를 생성하라.

### ❌ 플래그 중첩 (Nested Flags)

```typescript
// ❌ 하지 마라
if (isEnabled('flag_a')) {
  if (isEnabled('flag_b')) {
    // 조합 폭발 → 테스트 불가능
  }
}
```

## 정리

| 단계 | 핵심 |
|------|------|
| 코드 생성 | AI에게 Flag 포함 구현 요청 |
| PR 리뷰 | fallback 동작 검증 |
| 배포 | 1% 카나리아부터 시작 |
| 모니터링 | error rate, latency 비교 |
| 확대 | 단계별 10% → 50% → 100% |
| 정리 | 안정화 후 Flag 제거 |

---

*이 워크플로우는 텐빌더 채널에서 다루는 AI 코딩 도구 활용 시리즈의 일부입니다.*
*최신 정보는 [@ten-builder](https://youtube.com/@ten-builder) 채널을 구독해주세요.*
