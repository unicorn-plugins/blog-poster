# Blog Poster

> 기술/IT 블로그 글을 주제 리서치부터 최종 Word 문서 출력까지 자동화하는 블로그 작성 에이전트


---

## 개요

Blog Poster는 블로그 주제만 입력하면 리서치, 글 작성, SEO 최적화, 이미지 생성, Word 문서 출력까지 6단계 워크플로우를 자동화하는 DMAP 플러그인임.
4개의 전문 에이전트가 협업하여 한국어 기술 블로그 글을 빠르게 제작함.

**주요 기능:**
- 주제 키워드 기반 웹 검색, 기술 문서 조사, 트렌드 분석
- 리서치 기반 블로그 본문 작성 (친근한 캐주얼 톤, 1,000~2,000자)
- SEO 최적화 (키워드 분석, 메타 태그, 제목 최적화, SEO 점수 평가)
- AI 이미지 생성 (Gemini 기반 블로그 삽화/썸네일)
- 최종 Word(.docx) 문서 출력

---

## 설치

### 사전 요구사항

- [Claude Code](https://claude.com/claude-code) CLI 설치
- Node.js 18+ (context7 MCP 서버용, 선택)
- Python 3.10+ (이미지 생성, Word 문서 생성용)

### 플러그인 설치

**방법 1: 마켓플레이스 — GitHub (권장)**

```bash
# 1. GitHub 저장소를 마켓플레이스로 등록
claude plugin marketplace add {owner}/blog-poster

# 2. 플러그인 설치
claude plugin install blog-poster@blog-poster

# 3. 설치 확인
claude plugin list
```

**방법 2: 마켓플레이스 — 로컬**

```bash
# 1. 로컬 경로를 마켓플레이스로 등록
claude plugin marketplace add ./blog-poster

# 2. 플러그인 설치
claude plugin install blog-poster@blog-poster

# 3. 설치 확인
claude plugin list
```

> **설치 후 setup 스킬 실행:**
> ```
> /blog-poster:setup
> ```
> - `gateway/install.yaml`을 읽어 MCP 서버, 커스텀 도구 자동 설치
> - 설치 결과 검증 (`required: true` 항목 실패 시 중단)
> - 플러그인 활성화 확인 (스킬 자동 탐색)
> - 적용 범위 선택 (모든 프로젝트 / 현재 프로젝트만)

### 처음 GitHub을 사용하시나요?

다음 가이드를 참고하세요:

- [GitHub 계정 생성 가이드](https://github.com/cna-bootcamp/gen-ma-plugin/blob/main/resources/guides/github/github-account-setup.md)
- [Personal Access Token 생성 가이드](https://github.com/cna-bootcamp/gen-ma-plugin/blob/main/resources/guides/github/github-token-guide.md)
- [GitHub Organization 생성 가이드](https://github.com/cna-bootcamp/gen-ma-plugin/blob/main/resources/guides/github/github-organization-guide.md)

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

### 로컬 마켓플레이스

```bash
# 1. 로컬 플러그인 소스 갱신
cd ./blog-poster
git pull origin main

# 2. 마켓플레이스 업데이트
claude plugin marketplace update blog-poster

# 3. 플러그인 재설치
claude plugin install blog-poster@blog-poster
```

> **setup 재실행**: 업그레이드 후 `gateway/install.yaml`에 새 도구가 추가된 경우
> `/blog-poster:setup`을 재실행하여 누락된 도구를 설치할 것.

---

## 사용법

### 슬래시 명령

| 명령 | 설명 |
|------|------|
| `/blog-poster:setup` | 플러그인 초기 설정 (MCP, 도구, 의존성 설치) |
| `/blog-poster:help` | 사용 가능한 명령 및 워크플로우 안내 |
| `/blog-poster:write-post` | 블로그 글 작성 메인 워크플로우 (6단계) |

### 사용 예시

```
사용자: /blog-poster:write-post Kubernetes 오토스케일링 입문 가이드
→ 6단계 자동 진행: 리서치 → 초안 → SEO → 이미지 → Word 문서 출력
```

```
사용자: 블로그 써줘: React Server Components 장단점
→ core 스킬이 자동 감지 → write-post 워크플로우 시작
```

---

## 에이전트 구성

| 에이전트 | 티어 | 역할 |
|----------|------|------|
| researcher | MEDIUM | 웹 검색, 기술 문서 조사, 트렌드 분석 |
| writer | MEDIUM | 블로그 본문 작성, 구조화, Word 문서 생성 |
| seo-optimizer | MEDIUM | 키워드 분석, 메타 태그, SEO 점수 평가 |
| image-creator | MEDIUM | AI(Gemini) 기반 블로그 삽화/썸네일 생성 |

---

## 요구사항

### 필수 도구

| 도구 | 유형 | 용도 | 필수 |
|------|------|------|:----:|
| python-docx | Python 패키지 | Word(.docx) 문서 생성 | ✅ |
| context7 | MCP 서버 | 기술 문서 검색 (리서치 품질 향상) | 선택 |
| generate_image | 커스텀 도구 | AI 이미지 생성 (Gemini 기반) | 선택 |

### 환경 변수

| 변수 | 필수 | 설명 |
|------|:----:|------|
| GEMINI_API_KEY | 선택 | Google Gemini API Key (이미지 생성용) |

### 런타임 호환성

| 런타임 | 지원 |
|--------|:----:|
| Claude Code | ✅ |
| Codex CLI | 미검증 |
| Gemini CLI | 미검증 |

---

## 디렉토리 구조

```
blog-poster/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   ├── setup/
│   │   └── SKILL.md
│   ├── core/
│   │   └── SKILL.md
│   ├── help/
│   │   └── SKILL.md
│   └── write-post/
│       └── SKILL.md
├── agents/
│   ├── researcher/
│   │   ├── AGENT.md
│   │   ├── agentcard.yaml
│   │   └── tools.yaml
│   ├── writer/
│   │   ├── AGENT.md
│   │   ├── agentcard.yaml
│   │   └── tools.yaml
│   ├── seo-optimizer/
│   │   ├── AGENT.md
│   │   ├── agentcard.yaml
│   │   └── tools.yaml
│   └── image-creator/
│       ├── AGENT.md
│       ├── agentcard.yaml
│       └── tools.yaml
├── gateway/
│   ├── install.yaml
│   ├── runtime-mapping.yaml
│   ├── mcp/
│   │   └── context7.json
│   └── tools/
│       └── generate_image.py
├── commands/
│   ├── setup.md
│   ├── help.md
│   └── write-post.md
└── README.md
```

---

## 라이선스

MIT License

