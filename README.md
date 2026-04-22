# blog-poster

> AI 주제 검색부터 한국어 블로그 docx 산출까지 자동화하는 DMAP 플러그인

---

## 개요

blog-poster는 사용자가 지정한 주제를 웹·유튜브에서 병렬 검색하고, 수집된 정보를 바탕으로  
한국어 블로그 글을 초안 → 리뷰 → SEO 최적화 → 이미지 생성 → docx 산출 순서로 자동 작성함.  
기획자가 주제·독자·톤·분량만 입력하면 완성된 `.docx` 문서를 산출함.

**주요 기능:**
- 웹/유튜브 병렬 검색 및 리서치 종합 (`output/{주제}/0.research.md`)
- 독자·톤·분량 맞춤 한국어 블로그 초안 자동 작성
- 리뷰·SEO 최적화·AI 이미지 생성(최대 5개) 포함 전체 파이프라인
- python-docx 기반 `.docx` 산출 (이미지 임베드)

---

## 설치

### 사전 요구사항

- [Claude Code](https://claude.com/claude-code) CLI 설치
- Python 3.10+
- YouTube Data API v3 키 (`YOUTUBE_API_KEY`)
- Gemini API 키 (`GEMINI_API_KEY`)

### 플러그인 설치

**방법 1: 마켓플레이스 — GitHub (권장)**

```bash
# 1. GitHub 저장소를 마켓플레이스로 등록
claude plugin marketplace add dreamondal/blog-poster

# 2. 플러그인 설치
claude plugin install blog-poster@blog-poster

# 3. 설치 확인
claude plugin list
```

**방법 2: 마켓플레이스 — 로컬**

```bash
# 1. 로컬 경로를 마켓플레이스로 등록
claude plugin marketplace add /path/to/blog-poster

# 2. 플러그인 설치
claude plugin install blog-poster@blog-poster

# 3. 설치 확인
claude plugin list
```

**방법 3: 프로젝트 디렉토리에서만 동작하게 설치**
```bash
cd /path/to/blog-poster
claude --plugin-dir .
```

> **설치 후 setup 스킬 실행:**
> ```
> /blog-poster:setup
> ```
> - `gateway/install.yaml`을 읽어 필수 Python 의존성 자동 설치
> - API Key 환경 변수 설정 (`YOUTUBE_API_KEY`, `GEMINI_API_KEY`)
> - 모델 매핑 확인 및 런타임 어댑터 업데이트

---

## 업그레이드

### Git Repository 마켓플레이스

```bash
# 마켓플레이스 업데이트 (최신 커밋 반영)
claude plugin marketplace update blog-poster

# 플러그인 재설치
claude plugin install blog-poster@blog-poster

# 설치 확인
claude plugin list
```

> **갱신이 반영되지 않는 경우**: 플러그인을 삭제 후 재설치함.
> ```bash
> claude plugin remove blog-poster@blog-poster
> claude plugin marketplace update blog-poster
> claude plugin install blog-poster@blog-poster
> ```

> **setup 재실행**: 업그레이드 후 `gateway/install.yaml`에 새 도구가 추가된 경우
> `/blog-poster:setup`을 재실행하여 누락된 도구를 설치할 것.

---

## 사용법

### 슬래시 명령: 플러그인 설치 후 사용 가능

| 명령 | 설명 |
|------|------|
| `/blog-poster:setup` | 플러그인 초기 설정 (Python 의존성, API Key, 모델 매핑) |
| `/blog-poster:blog-post` | 주제 검색부터 docx 문서 산출까지 블로그 글 전체 작성 |
| `/blog-poster:help` | 사용 안내 (명령 목록, 자동 라우팅, Quick Guide) |

### @ 명령: 플러그인 디렉토리에서 바로 사용 가능

| 명령 | 설명 |
|------|------|
| `@setup` | 플러그인 초기 설정 |
| `@blog-post` | 블로그 글 전체 작성 파이프라인 실행 |
| `@help` | 사용 안내 |

### 사용 예시

```
사용자: @blog-post
→ 글 주제, 독자, 톤앤매너, 분량 수집
→ 웹/유튜브 병렬 검색 → 리서치 종합
→ 초안 작성 → 리뷰 → SEO 최적화 → 이미지 생성
→ output/{주제}/5.{주제}.docx 산출
```

---

## 에이전트 구성

| 에이전트 | 티어 | 역할 |
|----------|------|------|
| researcher | MEDIUM | 웹/유튜브 병렬 검색 및 콘텐츠 종합 |
| writer | MEDIUM | 블로그 초안 작성 및 SEO 최적화 |
| reviewer | HIGH | 가독성·정확성·구성 관점 품질 검토 |
| executor | MEDIUM | 이미지 대상 선별 및 generate_image 병렬 실행 |

---

## 요구사항

### 필수 도구

| 도구 | 유형 | 용도 |
|------|------|------|
| youtube_search.py | Custom | YouTube Data API v3 검색·자막 추출 |
| generate_image.py | Custom | Gemini Nano Banana 이미지 생성 |
| python-docx | Python 라이브러리 | docx 문서 빌드 |

### 환경 변수

| 변수명 | 용도 |
|--------|------|
| `YOUTUBE_API_KEY` | YouTube Data API v3 인증 |
| `GEMINI_API_KEY` | Gemini 이미지 생성 API 인증 |

### 런타임 호환성

| 런타임 | 지원 |
|--------|:----:|
| Claude Code | ✅ |
| Cursor | ✅ |
| Codex CLI | ✅ |
| Antigravity | ✅ |

---

## 디렉토리 구조

```
blog-poster/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── agents/
│   ├── researcher/
│   │   ├── AGENT.md
│   │   ├── agentcard.yaml
│   │   └── tools.yaml
│   ├── writer/
│   ├── reviewer/
│   └── executor/
├── skills/
│   ├── blog-post/
│   │   └── SKILL.md
│   ├── generate-docx/
│   │   ├── SKILL.md
│   │   ├── assets/
│   │   └── references/
│   ├── setup/
│   └── help/
├── gateway/
│   ├── install.yaml
│   ├── runtime-mapping.yaml
│   └── tools/
│       ├── youtube-search/
│       │   └── youtube_search.py
│       └── general/
│           └── generate_image.py
├── commands/
│   ├── blog-post.md
│   ├── setup.md
│   └── help.md
├── output/              # 산출물 저장 (gitignore 제외)
│   └── {주제}/
│       ├── 0.research.md
│       ├── 1.draft.md
│       ├── 2.revise.md
│       ├── 3.seo-optimize.md
│       ├── 4.final.md
│       ├── 5.{주제}.docx
│       └── images/
├── AGENTS.md
├── CLAUDE.md
└── README.md
```

---

## 라이선스

MIT License
