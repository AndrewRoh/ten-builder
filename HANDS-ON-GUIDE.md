# 텐빌더 핵심 워크플로 따라하기 가이드

> 설치는 됐다고 가정하고, 실무에서 바로 쓰는 6가지 핵심 워크플로를 직접 따라해봅니다.

**사전 조건:** Claude Code 설치 완료 (`claude --version` 으로 확인)

---

## 목차

| # | 워크플로 | 시간 |
|---|---------|------|
| 01 | [AI 코드 리뷰](#워크플로-01--ai-코드-리뷰) | 10분 |
| 02 | [AI 디버깅](#워크플로-02--ai-디버깅) | 15분 |
| 03 | [AI 리팩토링](#워크플로-03--ai-리팩토링) | 20분 |
| 04 | [MCP 도구 연결](#워크플로-04--mcp-도구-연결) | 15분 |
| 05 | [Hooks 자동화](#워크플로-05--hooks-자동화) | 20분 |
| 06 | [서브에이전트 병렬 실행](#워크플로-06--서브에이전트-병렬-실행) | 20분 |

---

## 워크플로 01 — AI 코드 리뷰

> PR 올리기 전 5분 만에 셀프 리뷰 완료하기

### 시나리오

새로운 기능을 구현하고 PR을 올리기 전입니다. 버그나 보안 이슈가 없는지 AI에게 먼저 검토시켜봅니다.

---

### Step 1. 현재 변경사항 확인

```bash
# 먼저 뭘 바꿨는지 확인
git diff --stat
```

**예시 출력:**
```
 src/api/auth.ts     | 45 ++++++++++++--
 src/middleware/jwt.ts | 23 +++++++
 tests/auth.test.ts   | 12 +++
```

---

### Step 2. AI 셀프 리뷰 실행

```bash
claude "현재 브랜치의 변경사항을 리뷰해줘.

다음 체크리스트로 확인해줘:
- 보안 취약점 (인증 우회, SQL 인젝션, XSS)
- 에러 핸들링 누락
- 타입 안전성 (TypeScript)
- 성능 이슈 (N+1, 불필요한 루프)
- 테스트 커버리지 부족

문제가 없는 항목은 간단히 ✅로 표시해줘.
문제가 있으면 파일명:라인 형태로 위치를 알려줘."
```

---

### Step 3. 발견된 문제 즉시 수정

AI가 문제를 발견했다면:

```bash
# 예: auth.ts 42번 줄에 null 체크 누락 발견
claude "auth.ts 42번 줄에 null 체크를 추가해줘.
기존 테스트가 깨지지 않도록 해줘."
```

수정 후 바로 테스트 확인:

```bash
npm test
```

---

### Step 4. PR 설명 자동 생성

```bash
claude "이 변경사항으로 PR 설명을 작성해줘.

포맷:
## 변경 내용
## 왜 이렇게 했나
## 테스트 방법"
```

---

### 커스텀 커맨드로 등록하면 더 편해요

매번 긴 프롬프트를 치기 귀찮다면, 슬래시 커맨드로 등록합니다.

```bash
mkdir -p .claude/commands
```

```markdown
# .claude/commands/review.md

현재 브랜치의 변경사항을 리뷰해줘.

체크리스트:
- 보안 취약점
- 에러 핸들링 누락
- 타입 안전성
- 성능 이슈

문제가 없으면 ✅ LGTM을 출력해줘.
```

이제부터 `/project:review`로 바로 실행됩니다.

---

### ✅ 체크포인트

- [ ] `claude "현재 브랜치 변경사항 리뷰해줘"` 실행해봤나요?
- [ ] AI가 발견한 문제를 수정하고 테스트 통과했나요?
- [ ] `.claude/commands/review.md` 만들어서 `/project:review` 실행해봤나요?

---

## 워크플로 02 — AI 디버깅

> 에러 메시지 붙여넣기 → 근본 원인 파악 → 수정까지 15분

### 시나리오

프로덕션에서 간헐적으로 500 에러가 발생하고 있습니다. 로그를 분석해서 원인을 찾아봅니다.

---

### Step 1. 에러 컨텍스트를 통째로 던지기

핵심은 **에러 메시지 + 재현 방법 + 예상 동작**을 함께 제공하는 것입니다.

```bash
claude "다음 에러가 발생합니다:

에러: TypeError: Cannot read properties of undefined (reading 'id')
스택: at getUserOrders (src/services/order.ts:52)
재현: POST /api/orders (userId가 없는 JWT 토큰으로 요청)
예상: 401 Unauthorized
실제: 500 Internal Server Error

코드를 수정하지 말고 원인 분석만 먼저 해줘."
```

> **팁:** 수정 전에 분석부터 요청하세요. 원인을 이해해야 올바른 수정이 가능합니다.

---

### Step 2. 빌드/테스트 에러를 파이프로 던지기

터미널 출력을 그대로 Claude에게 전달합니다.

```bash
# 빌드 에러
npm run build 2>&1 | claude "이 빌드 에러를 분석하고 수정해줘"

# 테스트 실패
npm test 2>&1 | claude "실패한 테스트를 분석하고 수정해줘"

# ESLint 에러
npx eslint src/ 2>&1 | claude "이 린트 에러를 전부 수정해줘"
```

---

### Step 3. 가설 수립 → 검증 → 수정

```bash
# 1. 원인 후보 나열
claude "이 에러의 원인 후보 3가지를 나열해줘.
각각 확인 방법과 수정 방법도 알려줘."

# 2. 가장 유력한 원인 확인
claude "1번 원인 (JWT 페이로드 null 체크 누락)을 확인해줘.
src/middleware/auth.ts를 확인해봐."

# 3. 수정 및 테스트
claude "원인이 맞으면 수정하고 해당 케이스에 대한 테스트 케이스도 추가해줘."
```

---

### Step 4. 간헐적 버그 추적

재현이 어려운 버그라면:

```bash
claude "이 간헐적 에러를 분석해줘.

로그 패턴:
- 하루 중 오전 9-10시에만 발생
- 특정 사용자(userId: 1234)에게서만 발생
- DB 응답 시간이 평소보다 300ms 느린 시점에 발생

로그 파일: logs/app-2026-04-09.log (첨부)"
```

---

### ✅ 체크포인트

- [ ] 에러를 `2>&1 | claude "..."` 파이프로 전달해봤나요?
- [ ] 수정 전에 분석을 먼저 요청하는 습관이 들었나요?
- [ ] 수정 후 테스트를 자동으로 추가했나요?

---

## 워크플로 03 — AI 리팩토링

> 레거시 코드를 안전하게, 단계별로 개선하기

### 시나리오

300줄짜리 서비스 파일이 있습니다. 너무 커졌고, 테스트도 어렵습니다. AI와 함께 단계별로 개선해봅니다.

---

### Step 1. 현황 분석 (수정 없이)

```bash
claude "src/services/payment.ts를 분석해줘.

다음 항목을 체크해줘:
- 함수별 라인 수와 복잡도
- 중복 코드 패턴
- 단일 책임 원칙 위반 여부
- 테스트하기 어려운 구조

코드는 절대 수정하지 말고 분석만 해줘."
```

---

### Step 2. 리팩토링 계획 수립

```bash
claude "분석 결과를 바탕으로 리팩토링 계획을 세워줘.

조건:
1. 각 단계는 독립적으로 커밋 가능해야 해
2. 기존 테스트가 항상 통과해야 해
3. 한 단계에서 최대 3개 파일만 수정할 것"
```

**좋은 계획 예시:**

```
1단계: validatePayment 함수 추출 → utils/payment-validator.ts
2단계: 에러 타입 정의 분리 → types/payment-errors.ts
3단계: PaymentService 클래스로 캡슐화
4단계: 단위 테스트 보강
```

---

### Step 3. 단계별 실행 + 테스트 확인

```bash
# 1단계 실행
claude "리팩토링 1단계를 실행해줘.
완료 후 npm test를 실행하고 결과를 알려줘."

# 테스트 통과 확인 후 커밋
git add -A && git commit -m "refactor: extract validatePayment utility"

# 2단계
claude "테스트 통과 확인했어. 2단계 진행해줘."

# 이후 반복...
```

> **황금률:** 각 단계마다 커밋. 테스트 실패하면 즉시 되돌리고 원인 파악.

---

### Step 4. 테스트 없는 코드 리팩토링 (특수 케이스)

```bash
# 리팩토링 전 먼저 특성 테스트 작성
claude "payment.ts의 현재 동작을 테스트하는 '특성 테스트'를 작성해줘.
엣지 케이스 포함해서 현재 동작을 그대로 문서화하는 테스트야.
수정 전에 실행해서 기준점으로 삼을 거야."

# 특성 테스트 통과 확인 후 리팩토링 시작
npm test
claude "특성 테스트가 통과했어. 이제 1단계 리팩토링 해줘."
```

---

### ✅ 체크포인트

- [ ] 수정 없이 분석 먼저 요청했나요?
- [ ] 계획을 3단계 이상으로 나눴나요?
- [ ] 각 단계마다 테스트 확인 후 커밋했나요?

---

## 워크플로 04 — MCP 도구 연결

> Claude Code에 GitHub, DB를 연결해서 실제 데이터와 대화하기

### 시나리오

Claude Code가 GitHub 이슈를 직접 읽고, 데이터베이스를 직접 조회하게 만들어봅니다.

---

### Step 1. GitHub MCP 서버 연결

```bash
# GitHub Personal Access Token 준비 (Settings → Developer settings → Tokens)
export GITHUB_TOKEN=ghp_your_token_here

# MCP 서버 추가
claude mcp add github \
  -e GITHUB_PERSONAL_ACCESS_TOKEN=$GITHUB_TOKEN \
  -- npx -y @modelcontextprotocol/server-github
```

연결 확인:

```bash
# Claude Code 실행 후 /mcp 명령으로 확인
claude
> /mcp
```

---

### Step 2. GitHub MCP 실제 사용

이제 Claude Code가 GitHub에 직접 접근합니다.

```bash
claude "이 레포의 open 이슈 목록을 가져와서 우선순위별로 정리해줘"

claude "이슈 #42의 내용을 읽고 해결을 위한 코드 변경사항을 구현해줘"

claude "지난 7일간 머지된 PR 목록과 주요 변경사항을 요약해줘"
```

---

### Step 3. PostgreSQL MCP 연결 (선택)

```bash
# 로컬 DB 연결
claude mcp add postgres \
  -- npx -y @modelcontextprotocol/server-postgres \
  "postgresql://username:password@localhost:5432/mydb"
```

실제 사용:

```bash
claude "users 테이블 구조를 보여주고, 최근 7일간 가입한 사용자 수를 알려줘"

claude "orders 테이블에서 환불 비율이 높은 상품 TOP 5를 찾아줘"
```

---

### Step 4. 프로젝트 MCP 설정 파일로 팀과 공유

```json
// .claude/settings.json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

이 파일을 Git에 커밋하면 팀 전체가 같은 MCP 설정을 사용합니다.  
(토큰은 `.env`에 별도 관리)

---

### ✅ 체크포인트

- [ ] `claude mcp add github ...` 으로 서버 추가했나요?
- [ ] `/mcp`로 연결 상태 확인했나요?
- [ ] GitHub 이슈를 Claude가 직접 읽게 해봤나요?
- [ ] `.claude/settings.json`에 MCP 설정 저장했나요?

---

## 워크플로 05 — Hooks 자동화

> 코드 수정할 때마다 자동으로 린트 실행, 위험 명령어 차단하기

### 시나리오

Claude Code가 파일을 수정할 때마다 자동으로 ESLint가 실행되고, 위험한 명령어는 실행 전에 차단되게 설정합니다.

---

### Step 1. settings.json 생성

```bash
mkdir -p .claude
touch .claude/settings.json
```

---

### Step 2. 파일 수정 후 자동 린트 Hook

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "npx eslint --fix $CLAUDE_FILE_PATH 2>/dev/null || true"
        }],
        "description": "파일 수정 후 자동 ESLint 실행"
      }
    ]
  }
}
```

테스트:

```bash
# Claude Code에서 파일 수정 요청
claude "src/index.ts에 console.log 하나 추가해줘"
# → 파일 수정 후 자동으로 ESLint 실행됨
```

---

### Step 3. 위험한 Bash 명령어 차단 Hook

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{
          "type": "command",
          "command": "node -e \"\nlet d='';\nprocess.stdin.on('data',c=>d+=c);\nprocess.stdin.on('end',()=>{\n  const i=JSON.parse(d);\n  const cmd=i.tool_input?.command||'';\n  const blocked=['rm -rf /','DROP TABLE','format c:'];\n  if(blocked.some(b=>cmd.includes(b))){\n    console.error('[Hook] BLOCKED: 위험한 명령어');\n    process.exit(2);\n  }\n  console.log(d);\n});\n\""
        }],
        "description": "위험한 명령어 차단"
      }
    ]
  }
}
```

---

### Step 4. 800줄 초과 파일 생성 차단 Hook

Claude가 너무 큰 파일을 만들지 않도록 제한합니다.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [{
          "type": "command",
          "command": "node -e \"\nlet d='';\nprocess.stdin.on('data',c=>d+=c);\nprocess.stdin.on('end',()=>{\n  const i=JSON.parse(d);\n  const content=i.tool_input?.content||'';\n  const lines=content.split('\\\\n').length;\n  if(lines>800){\n    console.error('[Hook] BLOCKED: '+lines+'줄 — 파일을 모듈로 분리하세요');\n    process.exit(2);\n  }\n  console.log(d);\n});\n\""
        }],
        "description": "800줄 초과 파일 차단"
      }
    ]
  }
}
```

---

### Step 5. 전체 settings.json 완성본

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [{
          "type": "command",
          "command": "node hooks/check-file-size.js"
        }],
        "description": "파일 크기 제한 (800줄)"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "npx eslint --fix $CLAUDE_FILE_PATH 2>/dev/null || true"
        }],
        "description": "자동 ESLint"
      }
    ]
  }
}
```

---

### ✅ 체크포인트

- [ ] `.claude/settings.json` 파일 만들었나요?
- [ ] 파일 수정 요청 후 ESLint가 자동 실행되나요?
- [ ] 위험한 명령어가 차단되는지 테스트했나요?

---

## 워크플로 06 — 서브에이전트 병렬 실행

> 하나의 프롬프트로 여러 작업을 동시에 처리하기

### 시나리오

새 API 모듈을 만들 때, 구현 + 테스트 + 문서 작성을 AI가 동시에 병렬로 처리하게 해봅니다.

---

### Step 1. 서브에이전트 기본 프롬프트 패턴

Claude Code는 하나의 요청 안에 여러 독립 작업이 있으면 자동으로 병렬 처리합니다.

```bash
claude "다음 3가지 작업을 병렬로 실행해줘:

1. src/services/notification.ts 구현
   - 이메일, SMS, 푸시 알림 전송 지원
   - 각 채널별 재시도 로직 포함

2. tests/notification.test.ts 작성
   - 각 알림 채널 성공/실패 케이스
   - 재시도 로직 검증

3. docs/notification-api.md 작성
   - API 사용법과 예제 코드 포함

각 작업은 독립적으로 처리하고 완료되면 보고해줘."
```

---

### Step 2. Git Worktree로 브랜치별 병렬 개발

서로 다른 기능을 브랜치별로 동시에 개발합니다.

```bash
# Worktree 2개 생성
git worktree add ../feature-auth feature/auth-module
git worktree add ../feature-payment feature/payment-module

# 터미널 1: 인증 모듈 개발
cd ../feature-auth
claude "JWT 기반 인증 모듈을 구현해줘"

# 터미널 2: (동시에) 결제 모듈 개발
cd ../feature-payment
claude "Stripe 연동 결제 모듈을 구현해줘"

# 완료 후 Worktree 정리
cd ~/myproject
git worktree remove ../feature-auth
git worktree remove ../feature-payment
```

---

### Step 3. 서브에이전트 프롬프트 잘 쓰는 법

좋은 서브에이전트 프롬프트의 조건:

```bash
# ❌ 나쁜 예 — 모호하고 의존성 있음
claude "API 만들고 테스트하고 배포해줘"

# ✅ 좋은 예 — 독립적이고 명확한 출력 형식
claude "다음 작업을 병렬로 실행해줘:

[작업 A — 독립적]
UserRepository 클래스를 src/repositories/user.ts에 구현해줘.
출력: 완성된 파일

[작업 B — 독립적]
UserRepository의 인터페이스를 types/repositories.ts에 정의해줘.
출력: 타입 정의 파일

[작업 C — 독립적]
UserRepository 모킹용 팩토리 함수를 tests/factories/user.ts에 작성해줘.
출력: 팩토리 함수 파일

모두 완료 후 각 파일의 주요 내용을 요약해줘."
```

---

### Step 4. 대규모 병렬 작업 예시 (모노레포)

```bash
claude "이 모노레포에서 다음 작업을 병렬로 처리해줘:

apps/web: TypeScript strict 모드 오류 수정
apps/api: 미사용 의존성 제거
packages/ui: 스토리북 컴포넌트 스냅샷 업데이트
packages/utils: 유닛 테스트 커버리지 80% 달성

각 패키지는 독립적으로 처리 가능해.
완료 순서대로 결과를 보고해줘."
```

---

### ✅ 체크포인트

- [ ] 3개 독립 작업을 하나의 프롬프트로 요청해봤나요?
- [ ] Git Worktree로 브랜치 2개를 동시에 만들어봤나요?
- [ ] 서브에이전트 프롬프트에 출력 형식을 명시했나요?

---

## 정리: 핵심 패턴 요약

| 상황 | 패턴 | 프롬프트 예시 |
|------|------|-------------|
| PR 전 검토 | 셀프 리뷰 | `"변경사항 리뷰해줘. 보안·에러·타입·성능 체크."` |
| 에러 발생 | 파이프 전달 | `npm test 2>&1 \| claude "이 에러 분석해줘"` |
| 코드 개선 | 분석 먼저 | `"수정하지 말고 분석만 먼저 해줘."` |
| 외부 데이터 | MCP 연결 | `claude mcp add github ...` |
| 반복 검사 | Hooks 설정 | PostToolUse에 ESLint 자동 실행 |
| 큰 작업 | 병렬 분해 | `"다음 3가지를 병렬로 실행해줘: ..."` |

---

## 자주 하는 실수 & 해결법

| 실수 | 문제 | 해결법 |
|------|------|--------|
| 범위가 너무 넓음 | "전체 리팩토링해줘" | 단계별로 나눠서 요청 |
| 수정부터 요청 | 원인 파악 없이 고침 | 분석 → 검증 → 수정 순서 |
| 테스트 없이 진행 | 리그레션 위험 | 각 단계마다 `npm test` |
| 컨텍스트 부족 | 엉뚱한 답변 | 파일명·줄번호·에러 메시지 포함 |
| 서브에이전트 의존성 | 작업 간 순서 의존 | 독립적인 작업만 병렬화 |

---

> 각 워크플로의 상세 가이드는 [COMPLETE-GUIDE.md](./COMPLETE-GUIDE.md) 를 참고하세요.
>
> **텐빌더** — [GitHub](https://github.com/AndrewRoh/ten-builder) · [뉴스레터](https://maily.so/tenbuilder) · [YouTube](https://youtube.com/@ten-builder)
