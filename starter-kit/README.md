# AI 코딩 스타터 킷

> 30분 안에 AI 코딩 환경을 세팅하는 가이드

## 이 폴더에 있는 것

| 파일/폴더 | 설명 |
|-----------|------|
| `macos-setup.sh` | macOS 원클릭 환경 설정 |
| `dotfiles/` | AI 코딩 도구 설정 파일 모음 |
| `workflows/` | 일일 AI 코딩 루틴 & PR 리뷰 워크플로 |

## Quick Start

```bash
# macOS 사용자
curl -sSL https://raw.githubusercontent.com/tenbuilder/tenbuilder/main/starter-kit/macos-setup.sh | bash

# 수동 설치
# 아래 단계별 가이드 참조
```

## 수동 설치

### 1. Claude Code 설치

```bash
npm install -g @anthropic-ai/claude-code
```

### 2. AI 코딩 도구 선택

| 도구 | 용도 | 설치 |
|------|------|------|
| **Claude Code** | CLI 기반 AI 코딩 | `npm i -g @anthropic-ai/claude-code` |
| **Cursor** | AI IDE | [cursor.sh](https://cursor.sh) |
| **GitHub Copilot** | 인라인 자동완성 | VS Code 확장 |

**추천 조합:** Claude Code (아키텍처/리팩토링) + Copilot (인라인 완성)

### 3. 설정 파일 복사

```bash
# Claude Code 설정
cp dotfiles/.claude/CLAUDE.md.example /your/project/CLAUDE.md

# Cursor 설정
cp dotfiles/.cursor/.cursorrules /your/project/.cursorrules

# AI 관련 alias
cat dotfiles/.zshrc.ai >> ~/.zshrc
source ~/.zshrc
```
