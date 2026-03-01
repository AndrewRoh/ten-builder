# 워크플로: MCP 서버 설정

> Claude Code에 외부 도구(GitHub, DB, 문서)를 연결하는 MCP 서버 설정 가이드

## MCP란?

**Model Context Protocol** — AI가 외부 도구와 소통하는 표준 프로토콜이에요. MCP 서버를 연결하면 Claude Code가 GitHub PR을 만들거나, DB를 조회하거나, 문서를 검색할 수 있어요.

## 설정 위치

| 범위 | 파일 | 용도 |
|------|------|------|
| 전역 | `~/.claude.json` | 모든 프로젝트에 적용 |
| 프로젝트 | `.claude/settings.json` | 해당 프로젝트만 |

## 추천 MCP 서버 5선

### 1. GitHub

PR 생성, 이슈 관리, 코드 리뷰를 Claude Code에서 직접 처리해요.

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_your_token"
      }
    }
  }
}
```

### 2. Supabase

데이터베이스 스키마 조회, 쿼리 실행, 마이그레이션을 연결해요.

```json
{
  "supabase": {
    "command": "npx",
    "args": ["-y", "@supabase/mcp-server-supabase@latest", "--project-ref=your_ref"]
  }
}
```

### 3. Context7

라이브러리 공식 문서를 실시간으로 조회해요. 오래된 지식 대신 최신 docs를 참조해요.

```json
{
  "context7": {
    "command": "npx",
    "args": ["-y", "@context7/mcp-server"]
  }
}
```

### 4. Vercel

배포 상태 확인, 로그 조회를 CLI 없이 처리해요.

```json
{
  "vercel": {
    "type": "http",
    "url": "https://mcp.vercel.com"
  }
}
```

### 5. Filesystem

특정 디렉토리의 파일을 AI가 직접 읽고 쓸 수 있어요.

```json
{
  "filesystem": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/projects"]
  }
}
```

## 설정 방법

### 전역 설정 (~/.claude.json)

```json
{
  "mcpServers": {
    "github": { "...": "..." },
    "context7": { "...": "..." }
  }
}
```

### 프로젝트별 비활성화

특정 프로젝트에서 필요 없는 서버를 끄려면:

```json
{
  "disabledMcpServers": ["filesystem", "supabase"]
}
```

## 주의사항

| 항목 | 설명 |
|------|------|
| **10개 이하 유지** | 활성 MCP 서버마다 스키마가 컨텍스트에 주입 → 토큰 소비 |
| **API 키 관리** | `.env` 또는 시스템 환경변수 사용 → JSON에 직접 넣지 마세요 |
| **서드파티 검증** | 공식 MCP 서버가 아니면 소스코드 확인 후 사용 |
| **HTTP vs stdio** | `type: "http"` = 원격 서버, `command` = 로컬 프로세스 |

## 트러블슈팅

| 증상 | 해결 |
|------|------|
| MCP 서버 연결 안 됨 | `npx -y <package>` 단독 실행으로 서버 테스트 |
| 토큰 에러 | API 키 유효성 확인, 권한 범위 (scope) 체크 |
| 느린 응답 | 불필요한 서버 비활성화, `disabledMcpServers` 활용 |
| 컨텍스트 부족 | MCP 서버 수 줄이기 (5개 이하 권장) |

---

📬 매주 AI 코딩 팁을 받아보세요 → [maily.so/tenbuilder](https://maily.so/tenbuilder)
