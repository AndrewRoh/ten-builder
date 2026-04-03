# 추론 모델(Reasoning Models) 코딩 활용 치트시트

> o3 · Claude Extended Thinking · Gemini 2.5 Pro — 한 장으로 보는 "생각하는 AI" 코딩 가이드

## 추론 모델이란?

응답 전에 **내부 추론(thinking) 단계**를 거쳐 복잡한 문제를 단계별로 해결하는 AI 모델. 일반 모델보다 느리고 비싸지만, 정확도가 높다.

## 2026 주요 추론 모델 비교

| 모델 | 제공사 | 컨텍스트 | Thinking 방식 | 코딩 강점 |
|------|--------|---------|--------------|----------|
| **o3** | OpenAI | 200K | Chain-of-Thought (숨김) | 알고리즘, 수학 |
| **o4-mini** | OpenAI | 200K | 경량 추론 | 빠른 추론 + 코딩 |
| **Claude Opus 4.6 + ET** | Anthropic | 1M | Extended Thinking (노출) | 대규모 코드베이스, 아키텍처 |
| **Claude Sonnet 4.5 + ET** | Anthropic | 200K | Extended Thinking | 균형잡힌 추론 코딩 |
| **Gemini 2.5 Pro** | Google | 1M+ | Deep Think | 긴 컨텍스트 분석 |
| **Gemini 2.5 Flash Thinking** | Google | 1M | 경량 추론 | 빠른 분석 |

## 언제 추론 모델을 쓰나?

### ✅ 추론 모델 사용

```
• 복잡한 알고리즘 설계 (DP, 그래프, 동시성)
• 다중 파일 의존성 디버깅
• 아키텍처 설계 결정
• 레이스 컨디션 / 데드락 분석
• 레거시 코드 역공학
• 보안 취약점 심층 분석
• 설계 트레이드오프 평가
```

### ❌ 일반 모델로 충분

```
• CRUD 엔드포인트 생성
• 보일러플레이트 코드
• 단순 UI 컴포넌트
• 변수/함수 이름 변경
• import 정리
• 단순 타입 에러 수정
• 테스트 추가 (단순 케이스)
```

## 도구별 추론 모드 활성화

### Claude Code

```bash
# Extended Thinking 활성화
claude --model claude-opus-4-6
# CLAUDE.md에서 thinking budget 설정
# thinking: high | medium | low

# Sonnet으로 빠른 추론
claude --model claude-sonnet-4-5
```

### OpenAI Codex CLI

```bash
# o3 풀 추론
codex --model o3

# o4-mini 경량 추론  
codex --model o4-mini
```

### Gemini CLI

```bash
# Deep Think 모드
gemini --model gemini-2.5-pro
```

## 추론 모델 프롬프트 패턴

### 패턴 1: 분석 → 계획 → 구현

```
"이 코드를 최적화해줘.
1단계: 현재 병목 지점 분석
2단계: 가능한 최적화 방안 3가지 비교
3단계: 최적 방안 구현"
```

### 패턴 2: 제약 조건 명시

```
"O(n log n) 시간, O(1) 추가 공간,
stable sort, in-place 정렬 구현"
```

### 패턴 3: 검증 요청

```
"구현 후 정확성을 검증해:
- 빈 입력
- 단일 요소
- 역순 정렬
- 중복 요소
- 최대 크기 (10^6)"
```

## 비용 비교 (코딩 작업 기준)

| 작업 유형 | 일반 모델 비용 | 추론 모델 비용 | 추론 가치 |
|----------|-------------|-------------|----------|
| CRUD 생성 | $0.01 | $0.05 | ❌ 낭비 |
| 알고리즘 구현 | $0.02 | $0.08 | ✅ 정확도↑ |
| 디버깅 (복잡) | $0.03 | $0.15 | ✅ 시간 절약 |
| 아키텍처 리뷰 | $0.05 | $0.20 | ✅ 품질↑ |
| 코드 리뷰 | $0.02 | $0.10 | ⚠️ 상황에 따라 |

## 비용 최적화 전략

```
1. 2단계 파이프라인: Haiku(초안) → Opus+ET(검증)
2. Thinking Budget 조절: 작업별 프리셋
3. 프롬프트 캐싱: 반복 컨텍스트 재사용
4. Batch API: 비실시간 작업 50% 할인
5. o4-mini: 추론 필요하지만 비용 민감할 때
```

## Extended Thinking의 특별한 장점

```
일반 모델:  입력 → [블랙박스] → 출력
추론 모델:  입력 → [사고 과정 노출] → 출력
                    ^^^^^^^^^^^^
              왜 이렇게 판단했는지 확인 가능
```

**Claude ET 활용 팁:**
- thinking 로그를 읽어 AI의 판단 근거 확인
- 잘못된 추론이 보이면 그 지점을 정정하고 재시도
- thinking 과정에서 발견한 엣지 케이스를 테스트에 반영

## Quick Decision: 어떤 모델?

```
복잡도 낮음 + 속도 중요 → Haiku / Flash
복잡도 중간 + 일반 코딩 → Sonnet / GPT-4o
복잡도 높음 + 정확성 중요 → Opus+ET / o3
긴 코드 분석 필요 → Opus+ET / Gemini 2.5 Pro (1M)
비용 민감 + 추론 필요 → o4-mini / Flash Thinking
```

---

*이 치트시트는 텐빌더 채널에서 다루는 AI 코딩 도구 활용 시리즈의 일부입니다.*
*최신 정보는 [@ten-builder](https://youtube.com/@ten-builder) 채널을 구독해주세요.*
