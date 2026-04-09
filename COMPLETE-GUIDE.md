# 텐빌더 완전 가이드

> AI로 10배 빠르게 빌드하는 방법 — 전체 통합 가이드 (2026.04 기준)

---

## 현재 AI 모델 버전 (2026년 4월 기준)

| 제공사 | 모델 | 모델 ID | 특징 |
|--------|------|---------|------|
| Anthropic | **Claude Opus 4.6** | `claude-opus-4-6` | 최고 성능, 복잡한 코딩/아키텍처 |
| Anthropic | **Claude Sonnet 4.6** | `claude-sonnet-4-6` | 균형잡힌 성능, 에이전틱 검색 |
| Anthropic | **Claude Haiku 4.5** | `claude-haiku-4-5` | 최고 속도, 간단한 태스크 |
| OpenAI | **GPT-4.1** | `gpt-4.1` | OpenAI 최신 모델 |
| Google | **Gemini 2.5 Pro** | `gemini-2.5-pro` | Google 최신 모델 |

---

## 목차

1. [환경 세팅](#1-환경-세팅)
2. [프로젝트 초기 설정](#2-프로젝트-초기-설정)
3. [일일 코딩 워크플로](#3-일일-코딩-워크플로)
4. [코드 리뷰 & 디버깅](#4-코드-리뷰--디버깅)
5. [리팩토링 & TDD](#5-리팩토링--tdd)
6. [MCP 도구 활용](#6-mcp-도구-활용)
7. [Hooks 자동화](#7-hooks-자동화)
8. [에이전트 팀 & 서브에이전트](#8-에이전트-팀--서브에이전트)
9. [백그라운드 에이전트 & Git Worktree](#9-백그라운드-에이전트--git-worktree)
10. [컨텍스트 엔지니어링](#10-컨텍스트-엔지니어링)
11. [프롬프트 엔지니어링](#11-프롬프트-엔지니어링)
12. [비용 최적화](#12-비용-최적화)
13. [보안 & 거버넌스](#13-보안--거버넌스)
14. [배포 & CI/CD 자동화](#14-배포--cicd-자동화)
15. [AI 코딩 도구 비교](#15-ai-코딩-도구-비교)
16. [팀 AI 도입 전략](#16-팀-ai-도입-전략)
17. [Skills & 커스텀 커맨드](#17-skills--커스텀-커맨드)
18. [실전 예제 & 워크플로 목록](#18-실전-예제--워크플로-목록)
19. [문서 변경 이력](#19-문서-변경-이력)

---

## 1. 환경 세팅

### 필수 설치

```bash
# Claude Code 설치
npm install -g @anthropic-ai/claude-code

# 설치 확인
claude --version
```

### 추천 도구 조합

| 용도 | 추천 도구 | 이유 |
|------|----------|------|
| 아키텍처/리팩토링 | **Claude Code** (CLI) | 터미널 기반, 자유도 최고 |
| 인라인 자동완성 | **GitHub Copilot** | VS Code 통합, 빠른 완성 |
| IDE 에이전트 | **Cursor** 또는 **Windsurf** | GUI 기반 에이전트 모드 |

### macOS 원클릭 설정

```bash
curl -sSL https://raw.githubusercontent.com/AndrewRoh/ten-builder/main/templates/macos-setup.sh | bash
```

### 수동 설정

```bash
# 1. CLAUDE.md 템플릿 복사
curl -O https://raw.githubusercontent.com/AndrewRoh/ten-builder/main/templates/CLAUDE.md.template
mv CLAUDE.md.template CLAUDE.md

# 2. Cursor 설정 (선택)
curl -O https://raw.githubusercontent.com/AndrewRoh/ten-builder/main/templates/cursorrules.template
mv cursorrules.template .cursorrules

# 3. AI shell alias (선택)
curl -O https://raw.githubusercontent.com/AndrewRoh/ten-builder/main/templates/.zshrc.ai
cat .zshrc.ai >> ~/.zshrc && source ~/.zshrc
```

**참고 가이드:** [환경 세팅 상세](./guides/1-environment-setup.md)

---

## 2. 프로젝트 초기 설정

### CLAUDE.md 구조

CLAUDE.md는 Claude Code에게 프로젝트 컨텍스트를 제공하는 핵심 파일입니다.

```markdown
# 프로젝트 개요
이 프로젝트는 [설명]입니다.

## 기술 스택
- Frontend: React 19 + TypeScript
- Backend: Node.js + Express
- DB: PostgreSQL + Prisma

## 코딩 컨벤션
- 함수명: camelCase
- 컴포넌트: PascalCase
- 파일명: kebab-case

## 자주 쓰는 명령어
- `npm run dev` — 개발 서버
- `npm test` — 테스트
- `npm run lint` — 린트

## 디렉토리 구조
src/
├── components/   # React 컴포넌트
├── hooks/        # 커스텀 훅
├── services/     # API 호출 로직
└── utils/        # 유틸리티 함수
```

### CLAUDE.md 최적화 팁

| 원칙 | 설명 |
|------|------|
| 간결하게 | 토큰 절약을 위해 핵심만 |
| 구체적으로 | "좋은 코드 써줘" 대신 컨벤션 명시 |
| 업데이트 | 프로젝트 변경 시 함께 갱신 |
| 계층화 | 루트 + 서브디렉토리별 CLAUDE.md |

**참고 가이드:** [프로젝트 초기 설정](./guides/2-project-setup.md) · [CLAUDE.md 최적화 플레이북](./claude-code/playbooks/13-claudemd-optimization.md)

---

## 3. 일일 코딩 워크플로

### 기본 루틴

```
아침: claude "오늘 작업할 이슈 정리해줘"
     → 이슈별 브랜치 생성 → 구현 시작

작업 중: claude "이 함수 리팩토링해줘"
        claude "이 에러 디버깅해줘"
        claude "테스트 작성해줘"

마무리: claude "오늘 변경사항 정리해서 PR 만들어줘"
```

### 효과적인 지시 패턴

```bash
# 나쁜 예 — 모호함
claude "코드 고쳐줘"

# 좋은 예 — 구체적
claude "src/auth/login.ts에서 비밀번호 검증 로직에 rate limiting을 추가해줘. 5분 내 5회 실패 시 계정 잠금."

# 더 좋은 예 — 컨텍스트 + 제약조건
claude "src/auth/login.ts의 validatePassword 함수를 수정해줘.
요구사항:
1. Redis 기반 rate limiting 추가
2. 5분 내 5회 실패 → 15분 잠금
3. 기존 테스트가 깨지지 않도록
4. 에러 메시지는 국제화(i18n) 키 사용"
```

**참고 가이드:** [일일 코딩 루틴](./guides/3-daily-workflow.md) · [스펙 기반 AI 개발](./guides/22-spec-driven-ai-development.md)

---

## 4. 코드 리뷰 & 디버깅

### AI 코드 리뷰

```bash
# Git diff 기반 리뷰
claude "git diff --staged를 리뷰해줘. 버그, 보안, 성능 이슈를 찾아줘."

# PR 리뷰
claude "이 PR의 변경사항을 리뷰해줘. 특히 에러 핸들링과 엣지 케이스를 중점적으로."
```

### 리뷰 체크리스트

| 항목 | 확인 내용 |
|------|----------|
| 로직 | 엣지 케이스, 경계값, null 처리 |
| 보안 | SQL 인젝션, XSS, 인증 우회 |
| 성능 | N+1 쿼리, 불필요한 렌더링 |
| 타입 | TypeScript 타입 안전성 |
| 테스트 | 새 코드에 대한 테스트 유무 |

### AI 디버깅

```bash
# 에러 메시지로 디버깅
claude "이 에러를 분석해줘: [에러 메시지 붙여넣기]"

# 로그 기반 디버깅
claude "이 로그를 분석해서 왜 API 응답이 느린지 찾아줘"

# 스크린샷 기반 디버깅 (멀티모달)
claude "이 스크린샷에서 UI 버그를 찾아줘" --image screenshot.png
```

### 디버깅 5단계 플로우

1. **재현** — 문제를 정확히 재현하는 최소 조건 확인
2. **격리** — 문제 범위를 좁히기 (이분법적 접근)
3. **진단** — 근본 원인 파악
4. **수정** — 최소한의 변경으로 해결
5. **검증** — 수정 후 기존 테스트 + 새 테스트 통과 확인

**참고 가이드:** [코드 리뷰](./guides/4-code-review.md) · [디버깅](./guides/5-debugging.md) · [AI 출력물 검증](./guides/18-ai-output-verification.md)

---

## 5. 리팩토링 & TDD

### AI 리팩토링 패턴

```bash
# 함수 분리
claude "이 300줄짜리 함수를 단일 책임 원칙에 맞게 분리해줘"

# 타입 마이그레이션
claude "이 모듈을 JavaScript에서 TypeScript로 마이그레이션해줘.
strict mode로 설정하고, any 사용 금지."

# 대규모 리팩토링
claude "이 모노리스 서비스를 마이크로서비스로 분리하기 위한 계획을 세워줘.
현재 의존성 그래프를 분석하고 분리 순서를 제안해줘."
```

### AI + TDD 워크플로 (Red-Green-Refactor)

```
1. Red   → claude "이 요구사항에 대한 실패하는 테스트를 먼저 작성해줘"
2. Green → claude "이 테스트를 통과하는 최소한의 구현을 해줘"
3. Refactor → claude "테스트가 통과하는 상태에서 코드를 개선해줘"
```

**참고 가이드:** [리팩토링](./guides/6-refactoring.md) · [TDD](./guides/7-tdd.md) · [AI + TDD 워크플로](./guides/49-ai-tdd-workflow.md)

---

## 6. MCP 도구 활용

### MCP(Model Context Protocol)란?

Claude Code가 외부 도구(GitHub, DB, Slack 등)에 직접 접근할 수 있게 해주는 프로토콜입니다.

### MCP 서버 추가

```bash
# GitHub MCP 서버
claude mcp add github npx -y @anthropic-ai/mcp-github

# PostgreSQL MCP 서버
claude mcp add postgres npx -y @anthropic-ai/mcp-postgres --connection-string "postgresql://..."

# Slack MCP 서버
claude mcp add slack npx -y @anthropic-ai/mcp-slack
```

### 주요 MCP 서버 생태계

| 카테고리 | MCP 서버 | 용도 |
|----------|---------|------|
| 코드 관리 | GitHub, GitLab | PR, 이슈, 코드 검색 |
| 데이터베이스 | PostgreSQL, MySQL, MongoDB | DB 조회/관리 |
| 커뮤니케이션 | Slack, Discord | 메시지 전송/검색 |
| 파일 시스템 | Filesystem | 파일 읽기/쓰기 |
| 검색 | Brave Search, Google | 웹 검색 |
| 모니터링 | Sentry, Datadog | 에러/성능 추적 |

### 커스텀 MCP 서버 만들기

```typescript
// my-mcp-server.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";

const server = new Server({ name: "my-tool", version: "1.0.0" }, {
  capabilities: { tools: {} }
});

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: "my_tool",
    description: "내 커스텀 도구",
    inputSchema: { type: "object", properties: { query: { type: "string" } } }
  }]
}));
```

**참고 가이드:** [MCP 도구](./guides/8-mcp-tools.md) · [커스텀 MCP 서버 워크플로](./workflows/custom-mcp-server.md)

---

## 7. Hooks 자동화

### Hooks란?

Claude Code의 도구 실행 전후에 자동으로 실행되는 셸/HTTP/LLM 트리거입니다.

### 설정 위치

```
~/.claude/settings.json          # 전역 설정
프로젝트/.claude/settings.json    # 프로젝트별 설정
```

### Hooks 설정 예시

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hook": "echo '명령어 실행 전 검사'",
        "description": "위험한 명령어 차단"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hook": "npx eslint --fix $CLAUDE_FILE_PATH",
        "description": "파일 수정 후 자동 린트"
      }
    ]
  }
}
```

### 실전 Hooks 패턴

| Hook | 트리거 | 용도 |
|------|--------|------|
| PreToolUse(Bash) | Bash 실행 전 | `rm -rf /` 같은 위험 명령어 차단 |
| PostToolUse(Write) | 파일 작성 후 | ESLint/Prettier 자동 실행 |
| PostToolUse(Edit) | 파일 수정 후 | 타입 체크, 유닛 테스트 실행 |
| Notification | 작업 완료 시 | Slack/Discord 알림 전송 |

**참고 가이드:** [Hooks](./guides/10-hooks.md) · [Hooks 치트시트](./cheatsheets/claude-code-hooks-cheatsheet.md)

---

## 8. 에이전트 팀 & 서브에이전트

### 에이전트 팀이란?

tmux를 사용해 여러 Claude Code 인스턴스를 동시에 실행하여 병렬 코딩하는 방식입니다.

### 에이전트 팀 실행

```bash
# 레포 클론
git clone https://github.com/AndrewRoh/ten-builder.git
cd ten-builder/episodes/ep5-agent-teams-with-tmux

# 미리보기
./run-agent-team.sh prompts --dry

# 실행 (tmux + Claude Code 필요)
./run-agent-team.sh prompts
```

### 5인 에이전트 팀 구조 예시

| 에이전트 | 역할 | 담당 |
|---------|------|------|
| Agent 1 | 레이아웃 | 전체 구조, 라우팅 |
| Agent 2 | 컴포넌트 | UI 컴포넌트 구현 |
| Agent 3 | 페이지 | 페이지별 로직 |
| Agent 4 | 데이터 | API 연동, 상태 관리 |
| Agent 5 | 스타일 | CSS, 테마, 반응형 |

### 서브에이전트 오케스트레이션

Claude Code 내장 서브에이전트는 메인 에이전트가 하위 태스크를 분할하여 병렬로 실행합니다.

```bash
# 서브에이전트 활용 프롬프트
claude "이 작업을 서브에이전트로 병렬 실행해줘:
1. src/components/ 전체 TypeScript 마이그레이션
2. src/services/ API 클라이언트 리팩토링  
3. src/utils/ 유닛 테스트 작성
각 서브에이전트는 독립적으로 작업하고, 결과를 머지해줘."
```

### 서브에이전트 격리 (Git Worktree)

```bash
# 서브에이전트는 Git Worktree를 사용해 독립된 작업 공간에서 실행
# 충돌 없이 병렬 개발 가능
```

**참고 가이드:** [에이전트 팀](./guides/11-agent-teams.md) · [서브에이전트 오케스트레이션](./guides/15-subagent-orchestration.md) · [멀티 에이전트 오케스트레이션](./guides/40-multi-agent-orchestration.md)

---

## 9. 백그라운드 에이전트 & Git Worktree

### 백그라운드 에이전트

Claude Code를 백그라운드에서 실행하여 비동기적으로 작업을 처리합니다.

```bash
# 헤드리스 모드로 백그라운드 실행
claude --headless "전체 테스트 스위트를 실행하고 실패한 테스트를 수정해줘"

# tmux 세션에서 실행
tmux new-session -d -s agent1 'claude "src/의 모든 TODO 코멘트를 처리해줘"'
```

### Git Worktree 병렬 개발

```bash
# Worktree 생성
git worktree add ../feature-auth feature/auth
git worktree add ../feature-api feature/api

# 각 Worktree에서 독립적으로 Claude Code 실행
cd ../feature-auth && claude "인증 모듈을 구현해줘"
cd ../feature-api && claude "API 엔드포인트를 구현해줘"

# 작업 완료 후 정리
git worktree remove ../feature-auth
git worktree remove ../feature-api
```

### 병렬 개발 워크플로

```
메인 브랜치 ─────────────────────────────── 머지
    ├── Worktree 1 (Agent A): 인증 모듈 ──┤
    ├── Worktree 2 (Agent B): API 개발 ───┤
    └── Worktree 3 (Agent C): 테스트 ─────┘
```

**참고 가이드:** [백그라운드 코딩 에이전트](./guides/46-background-coding-agents.md) · [Git Worktree 병렬 개발](./guides/48-git-worktree-ai-parallel-dev.md)

---

## 10. 컨텍스트 엔지니어링

### 컨텍스트 윈도우 관리

Claude Code는 최대 200K 토큰의 컨텍스트 윈도우를 사용합니다. 효율적인 관리가 중요합니다.

### 컨텍스트 최적화 전략

| 전략 | 방법 | 효과 |
|------|------|------|
| 계층적 CLAUDE.md | 루트 + 서브디렉토리별 | 필요한 컨텍스트만 로드 |
| 파일 참조 최소화 | 관련 파일만 명시적으로 지정 | 토큰 절약 |
| 요약 활용 | 큰 파일은 요약본 제공 | 핵심 정보 전달 |
| 세션 분리 | 태스크별 새 세션 | 누적 컨텍스트 방지 |

### 대규모 코드베이스 전략

```bash
# 나쁜 예 — 전체 코드베이스를 컨텍스트에 넣기
claude "전체 프로젝트를 분석해줘"

# 좋은 예 — 범위를 좁혀서 지시
claude "src/auth/ 디렉토리의 인증 로직만 분석해줘.
특히 JWT 토큰 갱신 로직에 집중해줘."
```

### 1M 컨텍스트 윈도우 활용 (Gemini 등)

대규모 컨텍스트가 필요한 경우 멀티 모델 전략을 사용합니다.

| 윈도우 크기 | 모델 | 적합한 작업 |
|------------|------|------------|
| ~200K | Claude Sonnet 4.6 | 일반 코딩, 리뷰 |
| ~200K | Claude Opus 4.6 | 복잡한 아키텍처 |
| ~1M | Gemini 2.5 Pro | 전체 코드베이스 분석 |

**참고 가이드:** [컨텍스트 엔지니어링](./guides/31-context-engineering.md) · [1M 컨텍스트 윈도우 전략](./guides/44-1m-context-window-strategy.md)

---

## 11. 프롬프트 엔지니어링

### 효과적인 프롬프트 구조

```
[역할] + [컨텍스트] + [구체적 지시] + [제약조건] + [출력 형식]
```

### 프롬프트 패턴 모음

| 패턴 | 프롬프트 예시 |
|------|-------------|
| 스펙 기반 | "이 OpenAPI 스펙에 맞춰 API를 구현해줘" |
| 예시 기반 | "기존 UserService처럼 OrderService를 만들어줘" |
| 제약 기반 | "React 19 + TypeScript strict 모드로 작성해줘" |
| 단계별 | "1단계: 인터페이스 정의 → 2단계: 구현 → 3단계: 테스트" |
| 비교 | "방법 A와 방법 B의 장단점을 비교하고 추천해줘" |

### 프롬프트 체이닝

복잡한 태스크를 단계별 프롬프트로 분해하는 기법입니다.

```
프롬프트 1: "이 요구사항을 분석하고 기술 스펙을 작성해줘"
    ↓
프롬프트 2: "이 스펙을 바탕으로 인터페이스를 설계해줘"
    ↓
프롬프트 3: "이 인터페이스를 구현해줘"
    ↓
프롬프트 4: "구현에 대한 테스트를 작성해줘"
    ↓
프롬프트 5: "코드 리뷰하고 개선해줘"
```

**참고 가이드:** [프롬프트 엔지니어링 치트시트](./cheatsheets/prompt-engineering-cheatsheet.md) · [프롬프트 체이닝 고급 패턴](./guides/50-advanced-prompt-chaining-patterns.md)

---

## 12. 비용 최적화

### 모델별 비용 비교 (API 기준)

| 모델 | Input (1M 토큰) | Output (1M 토큰) | 적합한 용도 |
|------|-----------------|------------------|------------|
| Claude Opus 4.6 | $15 | $75 | 아키텍처, 복잡한 로직 |
| Claude Sonnet 4.6 | $3 | $15 | 일반 코딩, 리뷰 |
| Claude Haiku 4.5 | $0.25 | $1.25 | 간단한 수정, 포맷팅 |

### 비용 절감 전략

| 전략 | 절감율 | 방법 |
|------|--------|------|
| 모델 라우팅 | 40-60% | 태스크 복잡도별 모델 선택 |
| 프롬프트 캐싱 | 50-90% | 반복 프롬프트 캐싱 활용 |
| 컨텍스트 최적화 | 20-30% | 불필요한 컨텍스트 제거 |
| 배치 처리 | 50% | Message Batches API 활용 |

### 모델 라우팅 가이드

```
간단한 작업 (포맷팅, 린트 수정) → Haiku 4.5
일반 코딩 (기능 구현, 버그 수정) → Sonnet 4.6
복잡한 작업 (아키텍처, 대규모 리팩토링) → Opus 4.6
```

### /cost 명령어로 비용 추적

```bash
# Claude Code 세션 비용 확인
/cost
# 캐시 히트 비율 포함 상세 비용 표시
```

**참고 가이드:** [비용 최적화](./guides/14-cost-optimization.md) · [AI 코딩 비용 최적화 치트시트](./cheatsheets/ai-coding-cost-optimization-cheatsheet.md)

---

## 13. 보안 & 거버넌스

### AI 코딩 보안 체크리스트

| 영역 | 확인 항목 |
|------|----------|
| 코드 유출 | 민감한 코드가 외부 API로 전송되지 않는지 |
| 시크릿 관리 | .env, API 키가 커밋되지 않는지 |
| 의존성 | AI가 추가한 패키지의 보안 검증 |
| 권한 | AI 에이전트의 파일/네트워크 접근 범위 |
| 감사 로그 | AI 에이전트 실행 기록 보존 |

### 보안 설정

```json
// .claude/settings.json
{
  "permissions": {
    "allow": ["Read", "Write", "Edit", "Glob", "Grep"],
    "deny": ["Bash(rm -rf *)"],
    "askUser": ["Bash"]
  }
}
```

### AI 에이전트 샌드박싱

| 방법 | 보안 수준 | 용도 |
|------|----------|------|
| 권한 설정 | 기본 | 일반 개발 |
| Docker 컨테이너 | 중간 | 격리 실행 |
| Git Worktree | 중간 | 병렬 작업 격리 |
| VM/원격 서버 | 높음 | 프로덕션급 격리 |

### MCP 프로덕션 보안

| 항목 | 설정 |
|------|------|
| 인증 | OAuth 2.0 / API 키 |
| 네트워크 | 허용된 도메인만 접근 |
| 로깅 | 모든 MCP 호출 기록 |
| 권한 | 최소 권한 원칙 적용 |

**참고 가이드:** [보안](./guides/9-security.md) · [AI 코딩 보안](./guides/16-ai-coding-security.md) · [AI 코드 보안 거버넌스](./guides/38-ai-code-security-governance.md)

---

## 14. 배포 & CI/CD 자동화

### AI 배포 워크플로

```bash
# PR 생성
claude "현재 변경사항으로 PR을 만들어줘.
제목, 설명, 리뷰어를 자동으로 설정해줘."

# CI 파이프라인 디버깅
claude "이 GitHub Actions 실패 로그를 분석하고 수정해줘"
```

### GitHub Actions + AI 리뷰

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
        run: |
          claude --headless "이 PR의 변경사항을 리뷰하고
          보안/성능/로직 이슈를 코멘트로 남겨줘"
```

### Pre-commit AI Hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: ai-review
        name: AI Quick Review
        entry: claude --headless "staged 변경사항에 명백한 버그가 있는지 확인해줘"
        language: system
        stages: [commit]
```

**참고 가이드:** [배포](./guides/12-deployment.md) · [GitHub Actions AI 리뷰](./workflows/github-actions-ai-review.md) · [Pre-commit AI Hooks](./workflows/pre-commit-ai-hooks.md)

---

## 15. AI 코딩 도구 비교

### 터미널 기반 AI 코딩 에이전트 (2026년 4월)

| 도구 | 모델 | 특징 | 가격 |
|------|------|------|------|
| **Claude Code** | Claude Opus/Sonnet 4.6 | 서브에이전트, Hooks, MCP, Git Worktree | $20/월 (Pro) |
| **Gemini CLI** | Gemini 2.5 Pro | 1M 컨텍스트, Google 생태계 | 무료 (제한) |
| **Aider** | 멀티모델 | 오픈소스, Git 네이티브 | 무료 + API 비용 |
| **OpenCode** | 멀티모델 | 오픈소스, 경량 | 무료 + API 비용 |
| **Cline** | 멀티모델 | VS Code 확장, 오픈소스 | 무료 + API 비용 |

### IDE 기반 AI 코딩 도구

| 도구 | 특징 | 가격 |
|------|------|------|
| **Cursor** | AI 네이티브 IDE, Agent 모드 | $20/월 (Pro) |
| **Windsurf** | Cascade 에이전트, 자동 컨텍스트 | $15/월 |
| **Kiro** (AWS) | 스펙 기반 에이전틱 IDE | 베타 |
| **GitHub Copilot** | VS Code 통합, 인라인 완성 | $10/월 |

### 도구 선택 가이드

```
"나는..." → 추천 도구
├── "터미널에서 작업하고 싶다" → Claude Code
├── "GUI IDE가 편하다" → Cursor
├── "예산이 제한적이다" → Aider + Claude API
├── "오픈소스만 쓴다" → OpenCode 또는 Aider
├── "대규모 코드베이스다" → Gemini CLI (1M 컨텍스트)
└── "팀 전체 도입이다" → Cursor 또는 GitHub Copilot
```

**참고 가이드:** [터미널 AI 에이전트 비교 2026](./guides/51-terminal-ai-agents-comparison-2026.md) · [AI CLI 도구 비교](./cheatsheets/ai-cli-tools-comparison.md)

---

## 16. 팀 AI 도입 전략

### 도입 단계

| 단계 | 기간 | 활동 |
|------|------|------|
| 1. 파일럿 | 2주 | 1-2명이 소규모 프로젝트에 적용 |
| 2. 확대 | 4주 | 팀의 50%로 확대, 가이드라인 수립 |
| 3. 전사 | 8주 | 전사 도입, 보안/거버넌스 체계 구축 |

### 팀 공유 설정

```
프로젝트 루트/
├── CLAUDE.md              # 팀 공유 프로젝트 컨텍스트
├── .claude/
│   ├── settings.json      # 팀 공유 설정
│   └── commands/          # 팀 공유 커스텀 커맨드
│       ├── review.md
│       ├── test-gen.md
│       └── deploy-check.md
└── .cursorrules           # Cursor 팀 설정 (선택)
```

### ROI 측정 지표

| 지표 | 측정 방법 |
|------|----------|
| 개발 속도 | PR 생성~머지 시간 변화 |
| 코드 품질 | 버그 발생률, 코드 리뷰 지적 사항 |
| 개발자 만족도 | 정기 설문 (NPS) |
| 비용 효율 | AI 도구 비용 vs 생산성 향상 |

**참고 가이드:** [팀 AI 도입](./guides/21-team-ai-adoption.md) · [AI 코딩 ROI 측정](./guides/28-ai-coding-roi-measurement.md)

---

## 17. Skills & 커스텀 커맨드

### 커스텀 커맨드

자주 쓰는 프롬프트를 슬래시 커맨드로 등록합니다.

```
.claude/commands/
├── review.md          # /project:review
├── test-gen.md        # /project:test-gen
├── refactor.md        # /project:refactor
└── deploy-check.md    # /project:deploy-check
```

### 커맨드 파일 예시

```markdown
# review.md
git diff --staged의 변경사항을 리뷰해줘.

확인 사항:
1. 로직 오류나 엣지 케이스
2. 에러 처리 누락
3. 타입 안전성
4. 불필요한 복잡도

문제가 없으면 "LGTM"만 출력해줘.
```

### $ARGUMENTS 파라미터

```markdown
# explain.md
$ARGUMENTS 파일의 코드를 분석해서 설명해줘.
```

```bash
# 사용
/project:explain src/auth/middleware.ts
```

### Claude Code Skills

Skills는 더 복잡한 워크플로를 패키지화한 확장입니다.

```bash
# Skills 설치 (텐빌더 제공)
curl -sSL https://raw.githubusercontent.com/AndrewRoh/ten-builder/main/skills/setup.sh | bash
```

| 스킬 | 명령어 | 설명 |
|------|--------|------|
| study-vault | `/study-vault` | PDF/문서를 학습 노트로 변환 |
| study-quiz | `/study-quiz` | 대화형 퀴즈로 숙달도 추적 |
| session-pack | `/pack` | 세션 종료 시 Memory/Handoff 자동 정리 |

**참고 가이드:** [커스텀 커맨드 치트시트](./cheatsheets/claude-code-commands-cheatsheet.md) · [Skills 아키텍처](./guides/30-ai-coding-skills-architecture.md) · [커스텀 룰 파일 설계](./guides/52-custom-rules-file-design.md)

---

## 18. 실전 예제 & 워크플로 목록

### 프로젝트 예제

| 카테고리 | 예제 | 설명 |
|----------|------|------|
| **풀스택** | [Next.js + Claude Code](./examples/nextjs-claude-code) | Next.js 프로젝트 AI 세팅 |
| | [Next.js AI 풀스택](./examples/nextjs-ai-fullstack) | 바이브 코딩으로 풀스택 앱 |
| | [Supabase + Next.js](./examples/supabase-nextjs-ai) | 풀스택 AI 개발 환경 |
| **백엔드** | [FastAPI + AI](./examples/fastapi-ai-testing) | FastAPI 테스트 |
| | [Express.js + AI](./examples/express-api-ai) | REST API 개발 |
| | [Go Microservice](./examples/go-microservice-ai) | Go 마이크로서비스 |
| | [Rust Axum](./examples/rust-axum-ai) | Rust API |
| | [GraphQL](./examples/graphql-ai-api) | GraphQL API |
| **프론트** | [React Native](./examples/react-native-ai) | 모바일 앱 |
| | [Chrome Extension](./examples/chrome-extension-ai) | 크롬 확장 |
| **DevOps** | [Terraform IaC](./examples/terraform-ai-iac) | 인프라 자동화 |
| | [Docker 환경](./workflows/docker-ai-dev-environment.md) | Docker AI 개발 |
| **봇** | [Discord 봇](./examples/discord-bot-ai) | Discord 봇 AI |
| | [Slack 봇](./examples/slack-bot-ai) | Slack 봇 AI |
| | [AI 코드 리뷰 봇](./examples/ai-code-review-bot) | 자동 코드 리뷰 |
| **에이전트** | [서브에이전트 병렬](./examples/subagent-parallel-dev) | 병렬 실행 |
| | [MCP 에이전트 도구킷](./examples/mcp-agent-toolkit) | MCP 3종 조합 |
| | [CrewAI 멀티 에이전트](./examples/crewai-multi-agent) | CrewAI 멀티 에이전트 |

### 자동화 워크플로

| 카테고리 | 워크플로 |
|----------|---------|
| **코드 품질** | [AI 코드 리뷰 자동화](./workflows/ai-code-review-automation.md) · [코드 품질 지표](./workflows/ai-code-quality-metrics.md) · [코드 거버넌스](./workflows/ai-code-governance.md) |
| **테스트** | [테스트 강화](./workflows/ai-test-augmentation.md) · [API 계약 테스트](./workflows/ai-api-contract-testing.md) · [마이그레이션 테스트](./workflows/ai-migration-test-pipeline.md) |
| **배포** | [GitHub Actions 리뷰](./workflows/github-actions-ai-review.md) · [Feature Flag](./workflows/ai-feature-flag-workflow.md) · [변경로그 자동화](./workflows/ai-changelog-automation.md) |
| **보안** | [의존성 감사](./workflows/ai-dependency-audit.md) · [서플라이 체인 감사](./workflows/ai-code-supply-chain-audit.md) · [프라이버시 컴플라이언스](./workflows/ai-privacy-compliance-pipeline.md) |
| **유지보수** | [기술 부채 해소](./workflows/ai-tech-debt-reduction.md) · [DB 마이그레이션](./workflows/ai-database-migration.md) · [레거시 코드 문서화](./workflows/ai-legacy-code-documentation.md) |
| **모니터링** | [에이전트 옵저버빌리티](./workflows/ai-agent-observability-pipeline.md) · [성능 프로파일링](./workflows/ai-performance-profiling.md) · [세션 메모리 관리](./workflows/ai-session-memory-management.md) |

---

## 19. 문서 변경 이력

### 2026-04-09 업데이트

이번 업데이트에서 다음 작업을 수행했습니다.

**모델명 업데이트 (12개 파일):**

| 파일 | 변경 내용 |
|------|----------|
| `cheatsheets/aider-cheatsheet.md` | claude-3-5-sonnet → claude-sonnet-4-6, claude-3-5-haiku → claude-haiku-4-5 |
| `cheatsheets/ai-cli-tools-comparison.md` | GPT-5.4 → GPT-4.1 |
| `cheatsheets/ai-model-routing-cheatsheet.md` | GPT-5/GPT-4o → GPT-4.1 |
| `cheatsheets/ai-coding-cost-optimization-cheatsheet.md` | Claude Haiku 3.5 → Claude Haiku 4.5 |
| `guides/14-cost-optimization.md` | gpt-5.x → gpt-4.1 |
| `guides/32-ai-agent-evaluation-framework.md` | gpt-5.2 → gpt-4.1 |
| `guides/35-ai-agent-scaffolding-design.md` | Claude 3.5 Sonnet → Claude Sonnet 4 |
| `guides/37-multimodal-ai-coding.md` | GPT-4o → GPT-4.1 |
| `guides/40-multi-agent-orchestration.md` | GPT-4o-mini → GPT-4.1-mini |
| `guides/43-llm-coding-workflow-optimization.md` | GPT-4o → GPT-4.1 |
| `guides/47-reasoning-models-coding.md` | GPT-4o → GPT-4.1 |
| `guides/54-data-driven-ai-tool-selection.md` | Claude 3.5 Sonnet → Claude Sonnet 4 |

**중복 문서 참고 사항:**

| 중복 그룹 | 파일 | 권장 |
|-----------|------|------|
| 백그라운드 에이전트 | guides/25, guides/46, guides/53 | 46을 메인으로 사용, 나머지는 참조용 |
| 에이전트 디버깅 치트시트 | ai-agent-debug-flow, ai-agent-debugging | 유사하나 관점이 다름 — 유지 |
| 가이드 번호 충돌 | guides/49 두 개 존재 | 49-ai-tdd-workflow를 56으로 리넘버링 권장 |

---

> **텐빌더** — AI로 10배 빠르게 빌드하는 방법을 알려드려요
>
> 뉴스레터: [maily.so/tenbuilder](https://maily.so/tenbuilder) · YouTube: [youtube.com/@ten-builder](https://youtube.com/@ten-builder)
