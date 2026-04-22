---
name: help
description: blog-poster 플러그인 사용 안내 (명령 목록, 자동 라우팅, Quick Guide)
type: utility
user-invocable: true
---

# Help

[HELP 활성화]

## 목표

사용자 도움말 제공

## 활성화 조건

사용자가 `/blog-poster:help` 호출 시 또는 "blog-poster 도움말", "blog-poster 뭘 할 수 있어" 키워드 감지 시.

## 사용 안내

**중요: 추가적인 파일 탐색이나 에이전트 위임 없이, 아래 내용을 즉시 사용자에게 출력하세요.**

### 명령어

| 명령 | 설명 |
|------|------|
| `/blog-poster:setup` | 플러그인 초기 설정 (의존성 설치, API Key 등록, 모델 확인) |
| `/blog-poster:help` | 사용 안내 (이 화면) |
| `/blog-poster:blog-post` | **블로그 글 작성** — 주제 입력부터 docx 문서 산출까지 전체 파이프라인 실행 |

`@{skill-name}`으로 Skill 직접 호출 가능

### 자동 라우팅 (키워드 감지)

- "블로그 글 작성", "블로그 포스팅", "AI 주제 블로그" → `/blog-poster:blog-post`
- "blog-poster 설치", "설정" → `/blog-poster:setup`
- "도움말", "뭘 할 수 있어" → `/blog-poster:help`

### Quick Guide

1. **최초 1회**: `/blog-poster:setup` — Python 의존성 설치, API Key(YouTube + Gemini) 등록
2. **블로그 작성**: `/blog-poster:blog-post` — 주제 입력 → 자동 리서치 → 초안 → 리뷰 → SEO → 이미지 → docx

### 산출물 디렉토리 구조

```
output/{주제}/
├── 0.research.md       # 웹/유튜브 리서치 결과
├── 1.draft.md          # 블로그 초안
├── 2.revise.md         # 리뷰/수정본
├── 3.seo-optimize.md   # SEO 최적화본
├── 4.final.md          # 이미지 삽입 최종본
├── 5.{주제}.docx       # 최종 Word 문서
└── images/             # 생성된 이미지
    ├── image_001.png
    └── ...
```
