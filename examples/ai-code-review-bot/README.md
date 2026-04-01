# AI 코드 리뷰 봇 구축 실전 예제

> GitHub PR에 자동으로 AI 코드 리뷰를 달아주는 봇 — Claude API + GitHub Actions 완전 구현

## 이 예제에서 배울 수 있는 것

- GitHub Actions에서 PR diff를 추출하고 AI API에 보내는 파이프라인 구축
- Claude API로 코드 리뷰 코멘트를 생성하는 프롬프트 엔지니어링
- 리뷰 결과를 PR 코멘트와 인라인 리뷰로 자동 게시하는 방법

## 프로젝트 구조

```
ai-code-review-bot/
├── .github/
│   └── workflows/
│       └── ai-review.yml        # GitHub Actions 워크플로우
├── scripts/
│   ├── review.ts                # 메인 리뷰 스크립트
│   ├── diff-parser.ts           # PR diff 파싱
│   ├── ai-reviewer.ts           # Claude API 호출
│   └── github-commenter.ts      # PR 코멘트 생성
├── prompts/
│   ├── system.md                # 시스템 프롬프트
│   ├── security.md              # 보안 리뷰 프롬프트
│   └── performance.md           # 성능 리뷰 프롬프트
├── package.json
├── tsconfig.json
└── README.md
```

## 핵심 구현

### 1. GitHub Actions 워크플로우

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  pull-requests: write
  contents: read

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - run: npm ci
      
      - name: Get PR Diff
        id: diff
        run: |
          git diff origin/${{ github.base_ref }}...HEAD \
            --unified=5 \
            -- '*.ts' '*.tsx' '*.js' '*.jsx' '*.py' \
            > /tmp/pr-diff.patch
          echo "lines=$(wc -l < /tmp/pr-diff.patch)" >> $GITHUB_OUTPUT
      
      - name: AI Review
        if: steps.diff.outputs.lines > 0
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
        run: npx tsx scripts/review.ts /tmp/pr-diff.patch
```

### 2. Diff 파서

```typescript
// scripts/diff-parser.ts
interface FileDiff {
  filename: string;
  additions: { line: number; content: string }[];
  deletions: { line: number; content: string }[];
  hunks: string[];
}

export function parseDiff(diffContent: string): FileDiff[] {
  const files: FileDiff[] = [];
  const fileBlocks = diffContent.split(/^diff --git/m).filter(Boolean);
  
  for (const block of fileBlocks) {
    const filenameMatch = block.match(/^.*b\/(.+)$/m);
    if (!filenameMatch) continue;
    
    const filename = filenameMatch[1];
    const additions: FileDiff['additions'] = [];
    const deletions: FileDiff['deletions'] = [];
    const hunks: string[] = [];
    
    let currentLine = 0;
    const lines = block.split('\n');
    
    for (const line of lines) {
      const hunkMatch = line.match(/^@@ -\d+(?:,\d+)? \+(\d+)/);
      if (hunkMatch) {
        currentLine = parseInt(hunkMatch[1]);
        hunks.push(line);
        continue;
      }
      
      if (line.startsWith('+') && !line.startsWith('+++')) {
        additions.push({ line: currentLine, content: line.slice(1) });
        currentLine++;
      } else if (line.startsWith('-') && !line.startsWith('---')) {
        deletions.push({ line: currentLine, content: line.slice(1) });
      } else if (!line.startsWith('\\')) {
        currentLine++;
      }
    }
    
    files.push({ filename, additions, deletions, hunks });
  }
  
  return files;
}
```

### 3. AI 리뷰어 (Claude API)

```typescript
// scripts/ai-reviewer.ts
import Anthropic from '@anthropic-ai/sdk';
import { readFileSync } from 'fs';

const client = new Anthropic();

interface ReviewComment {
  filename: string;
  line: number;
  severity: 'critical' | 'warning' | 'suggestion' | 'praise';
  category: 'security' | 'performance' | 'logic' | 'style' | 'test';
  comment: string;
}

interface ReviewResult {
  summary: string;
  comments: ReviewComment[];
  score: number; // 1-10
}

export async function reviewCode(
  diff: string,
  context?: string
): Promise<ReviewResult> {
  const systemPrompt = readFileSync('prompts/system.md', 'utf-8');
  
  const response = await client.messages.create({
    model: 'claude-sonnet-4-5-20250514',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [{
      role: 'user',
      content: `다음 코드 변경을 리뷰해주세요.

## PR Diff
\`\`\`diff
${diff}
\`\`\`

${context ? `## 추가 컨텍스트\n${context}` : ''}

## 응답 형식
JSON으로 응답해주세요:
{
  "summary": "전체 리뷰 요약 (3-5문장)",
  "comments": [
    {
      "filename": "파일경로",
      "line": 줄번호,
      "severity": "critical|warning|suggestion|praise",
      "category": "security|performance|logic|style|test",
      "comment": "구체적인 리뷰 코멘트"
    }
  ],
  "score": 8
}`
    }]
  });

  const text = response.content[0].type === 'text' 
    ? response.content[0].text : '';
  
  // JSON 블록 추출
  const jsonMatch = text.match(/\{[\s\S]*\}/);
  if (!jsonMatch) throw new Error('Failed to parse review response');
  
  return JSON.parse(jsonMatch[0]) as ReviewResult;
}
```

### 4. 시스템 프롬프트

```markdown
<!-- prompts/system.md -->
# AI Code Reviewer

당신은 시니어 소프트웨어 엔지니어로서 코드 리뷰를 수행합니다.

## 리뷰 원칙

1. **건설적**: 문제를 지적할 때는 반드시 개선 방안을 함께 제시
2. **구체적**: "이 부분이 좋지 않습니다" 대신 정확한 이유와 대안 설명
3. **우선순위**: critical > warning > suggestion 순으로 중요도 분류
4. **칭찬**: 잘된 부분도 언급하여 동기 부여

## 리뷰 관점

### 보안 (Security)
- SQL 인젝션, XSS, CSRF 취약점
- 하드코딩된 시크릿이나 API 키
- 부적절한 인증/인가 처리
- 입력 검증 누락

### 성능 (Performance)
- N+1 쿼리 패턴
- 불필요한 재렌더링 (React)
- 메모리 누수 가능성
- 비효율적인 알고리즘

### 로직 (Logic)
- 엣지 케이스 미처리
- 에러 핸들링 누락
- 레이스 컨디션
- null/undefined 미처리

### 스타일 (Style)
- 네이밍 컨벤션 일관성
- 코드 중복
- 불필요한 복잡성
- 매직 넘버/문자열

### 테스트 (Test)
- 테스트 커버리지 부족
- 엣지 케이스 테스트 누락
- 모킹 과다 사용
- 테스트 가독성

## 주의사항
- 5개 이하의 파일 변경: 파일당 최대 3개 코멘트
- 5개 초과 파일 변경: 가장 중요한 10개 코멘트만
- 단순 포매팅 변경은 무시
- 자동 생성 파일(lock 파일 등)은 무시
```

### 5. GitHub 코멘트 게시

```typescript
// scripts/github-commenter.ts
import { Octokit } from '@octokit/rest';

const octokit = new Octokit({ auth: process.env.GITHUB_TOKEN });

interface ReviewComment {
  filename: string;
  line: number;
  severity: string;
  category: string;
  comment: string;
}

const SEVERITY_EMOJI: Record<string, string> = {
  critical: '🔴',
  warning: '🟡',
  suggestion: '🔵',
  praise: '🟢',
};

export async function postReview(
  owner: string,
  repo: string,
  prNumber: number,
  summary: string,
  comments: ReviewComment[],
  score: number
) {
  // 1. 전체 요약 코멘트
  const summaryBody = `## 🤖 AI Code Review

### 종합 점수: ${score}/10 ${'⭐'.repeat(Math.round(score / 2))}

${summary}

### 리뷰 요약
| 유형 | 개수 |
|------|------|
| 🔴 Critical | ${comments.filter(c => c.severity === 'critical').length} |
| 🟡 Warning | ${comments.filter(c => c.severity === 'warning').length} |
| 🔵 Suggestion | ${comments.filter(c => c.severity === 'suggestion').length} |
| 🟢 Praise | ${comments.filter(c => c.severity === 'praise').length} |

---
*Powered by Claude API • [텐빌더](https://youtube.com/@ten-builder)*`;

  // 2. 인라인 코멘트 생성
  const reviewComments = comments
    .filter(c => c.severity !== 'praise')
    .map(c => ({
      path: c.filename,
      line: c.line,
      body: `${SEVERITY_EMOJI[c.severity]} **[${c.category}]** ${c.comment}`,
    }));

  // 3. PR Review 생성
  await octokit.pulls.createReview({
    owner,
    repo,
    pull_number: prNumber,
    body: summaryBody,
    event: comments.some(c => c.severity === 'critical') 
      ? 'REQUEST_CHANGES' 
      : 'COMMENT',
    comments: reviewComments,
  });
}
```

### 6. 메인 스크립트

```typescript
// scripts/review.ts
import { readFileSync } from 'fs';
import { parseDiff } from './diff-parser';
import { reviewCode } from './ai-reviewer';
import { postReview } from './github-commenter';

async function main() {
  const diffPath = process.argv[2];
  if (!diffPath) {
    console.error('Usage: tsx review.ts <diff-file>');
    process.exit(1);
  }

  const diff = readFileSync(diffPath, 'utf-8');
  const files = parseDiff(diff);
  
  console.log(`📝 Reviewing ${files.length} files...`);
  
  // 대규모 PR은 파일 단위로 분할 리뷰
  const MAX_DIFF_SIZE = 50000; // 50KB
  let reviewResult;
  
  if (diff.length > MAX_DIFF_SIZE) {
    console.log('📦 Large PR detected, splitting review...');
    const chunks = chunkFiles(files, MAX_DIFF_SIZE);
    const results = await Promise.all(
      chunks.map(chunk => reviewCode(chunk))
    );
    reviewResult = mergeResults(results);
  } else {
    reviewResult = await reviewCode(diff);
  }

  console.log(`✅ Review complete: score ${reviewResult.score}/10`);
  console.log(`📝 ${reviewResult.comments.length} comments`);

  // GitHub에 게시
  const [owner, repo] = (process.env.GITHUB_REPOSITORY || '').split('/');
  const prNumber = parseInt(process.env.PR_NUMBER || '0');
  
  if (owner && repo && prNumber) {
    await postReview(
      owner, repo, prNumber,
      reviewResult.summary,
      reviewResult.comments,
      reviewResult.score
    );
    console.log('📮 Review posted to GitHub');
  }
}

main().catch(console.error);
```

## 설정 가이드

### 1. GitHub Secrets 설정

```
Repository Settings → Secrets and variables → Actions

ANTHROPIC_API_KEY: sk-ant-...  (Claude API 키)
```

### 2. 패키지 설치

```json
{
  "name": "ai-code-review-bot",
  "private": true,
  "scripts": {
    "review": "tsx scripts/review.ts"
  },
  "dependencies": {
    "@anthropic-ai/sdk": "^0.30.0",
    "@octokit/rest": "^21.0.0",
    "tsx": "^4.0.0"
  },
  "devDependencies": {
    "typescript": "^5.5.0",
    "@types/node": "^22.0.0"
  }
}
```

### 3. TypeScript 설정

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "dist"
  },
  "include": ["scripts/**/*.ts"]
}
```

## 커스터마이징 옵션

### 리뷰 범위 조절

```yaml
# 특정 파일만 리뷰
- name: Get PR Diff
  run: |
    git diff origin/main...HEAD \
      -- '*.ts' '*.tsx' \
      ':!*.test.ts' ':!*.spec.ts' \
      ':!**/generated/**' \
      > /tmp/pr-diff.patch
```

### 모델 선택

```typescript
// 리뷰 복잡도에 따라 모델 변경
const model = diffLines > 500 
  ? 'claude-opus-4-6-20250414'    // 대규모 PR
  : 'claude-sonnet-4-5-20250514'; // 일반 PR
```

### 리뷰 관점 추가

```markdown
<!-- prompts/security.md -->
보안 전문가 관점으로 추가 리뷰:
- OWASP Top 10 체크
- 입력 검증 패턴 확인
- 암호화/해싱 적절성
```

## 비용 예상

| PR 규모 | Diff 크기 | 토큰 수 | 예상 비용 |
|---------|----------|---------|----------|
| 소형 (1-3 파일) | ~5KB | ~3K | $0.01 |
| 중형 (5-10 파일) | ~20KB | ~10K | $0.04 |
| 대형 (20+ 파일) | ~100KB | ~50K | $0.20 |

**월 100 PR 기준**: $2-10 (Sonnet 사용 시)

## 주의사항

- AI 리뷰는 **보조 수단**이지 사람 리뷰를 대체하지 않는다
- Critical 이슈는 반드시 사람이 확인
- 비즈니스 로직 검증은 도메인 전문가가 해야 한다
- API 키를 코드에 하드코딩하지 말 것

---

*이 예제는 텐빌더 채널에서 다루는 AI 코딩 도구 활용 시리즈의 일부입니다.*
*최신 정보는 [@ten-builder](https://youtube.com/@ten-builder) 채널을 구독해주세요.*
