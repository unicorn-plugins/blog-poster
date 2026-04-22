# 개발 계획서

## 기본 정보
- 플러그인명: blog-poster
- 목표: 특정 주제에 대한 정보를 검색하여 블로그 글을 작성
- 대상 도메인: 콘텐츠 제작
- 대상 사용자: 기획자
- 작성자: dreamondal

---

## 핵심기능

### 정보검색
Claude Code 내장 WebSearch로 공신력 있는 웹 소스(뉴스·공식 문서·학술 자료)를 선별하여
사용자 지정 AI 주제를 검색하고, youtube-search 도구(YouTube Data API v3)로 자막 있는
상위 영상 Top5를 선별·추출함. 웹/유튜브 검색은 병렬 수행하며 결과를
`output/{주제}/0.research.md`에 종합 저장함.

### 블로그글 작성
사용자가 지정한 주제·독자·톤앤매너·분량 기준으로 한국어 블로그 글을 작성함.
초안 → 리뷰/수정 → SEO 최적화 → 이미지 생성(병렬) → docx 빌드 순서로 진행하며,
생성된 이미지를 docx 본문에 임베드하여 최종 문서를 산출함.

---

## 업무 플로우

### 옵션 수집
AskUserQuestion으로 아래 항목을 수집함:
- 글 주제 (예: "Claude AI 활용법")
- 블로그 독자 (예: "IT 비전공 직장인")
- 글의 톤앤매너 (예: "친근하고 쉽게")
- 글의 분량 (예: "2000자")

### 정보검색
- Step 1. 웹/유튜브 검색 병렬 수행
  - 웹검색: researcher 에이전트의 web-search 세부역할 — WebSearch로 최소 5개 공신력 소스 수집
  - 유튜브검색: researcher 에이전트의 youtube-search 세부역할 — youtube_search.py로 자막 있는 Top5 선별
- Step 2. 콘텐츠 종합: 수집 정보를 구조화하여 `output/{주제}/0.research.md` 저장

### 블로그글 작성
- Step 1. 초안 작성: writer 에이전트(draft-writer) → `output/{주제}/1.draft.md`
- Step 2. 리뷰/업데이트: reviewer 에이전트 검토 후 writer 에이전트 반영 → `output/{주제}/2.revise.md`
- Step 3. SEO 최적화: writer 에이전트(seo-optimizer) → `output/{주제}/3.seo-optimize.md`
- Step 4. 이미지 생성: executor 에이전트(이미지 대상 선별 + generate_image.py 병렬 실행)
  → 이미지: `output/{주제}/images/`, 최종본: `output/{주제}/4.final.md`
- Step 5. docx 빌드: generate-docx 스킬 → `output/{주제}/5.{주제}.docx`

---

## 기술 요구사항

### 기술 스택
| 구분 | 내용 |
|------|------|
| 런타임 | Claude Code |
| 언어 | Python 3.10+ |
| docx 빌더 | python-docx |
| 이미지 생성 | Gemini Nano Banana (generate_image.py) |
| 유튜브 검색 | YouTube Data API v3 (youtube_search.py) |
| 웹 검색 | Claude Code 내장 WebSearch |

### 환경 변수 (`.env`)
| 변수명 | 용도 | 위치 |
|--------|------|------|
| YOUTUBE_API_KEY | YouTube Data API v3 키 | gateway/tools/.env |
| GEMINI_API_KEY | Gemini 이미지 생성 API 키 | gateway/tools/.env |

---

## 공유자원

| 자원 유형 | 자원명 | 자원 경로 | 복사 위치 |
|----------|--------|-----------|-----------|
| 도구 | youtube-search | C:/Users/hiond/plugins/dmap/resources/tools/customs/youtube-search/youtube_search.py | gateway/tools/youtube-search/ |
| 도구 | generate_image | C:/Users/hiond/plugins/dmap/resources/tools/customs/general/generate_image.py | gateway/tools/general/ |
| 가이드 | docx-build-guide | C:/Users/hiond/plugins/dmap/resources/guides/office/docx-build-guide.md | skills/generate-docx/references/ |
| 템플릿 | docx-builder-SKILL | C:/Users/hiond/plugins/dmap/resources/templates/office/docx-builder-SKILL.md | skills/generate-docx/assets/ |
| 샘플 | docx-build-sample | C:/Users/hiond/plugins/dmap/resources/samples/office/docx-build-sample.py | skills/generate-docx/references/ |

### 커스텀 도구 개발 계획
공유자원에 있는 도구(youtube-search, generate_image)를 그대로 활용하며 추가 개발 없음.

---

## 플러그인 구조 설계

---

### 에이전트 구성 설계

#### 에이전트 목록 및 역할

| 에이전트 | 티어 | 역할 | 책임 |
|---------|------|------|------|
| researcher | MEDIUM | 정보 수집 전문가 | 웹/유튜브 병렬 검색 및 콘텐츠 종합 |
| writer | MEDIUM | 블로그 작성 전문가 | 초안 작성 + SEO 최적화 |
| reviewer | HIGH | 품질 검토 전문가 | 가독성·정확성·구성 관점 리뷰 |
| executor | MEDIUM | 이미지 생성 전문가 | 이미지 대상 선별 + generate_image 병렬 실행 |

**세부역할(sub_roles):**

| 에이전트 | 세부역할 | 설명 |
|---------|---------|------|
| researcher | web-search | WebSearch로 공신력 있는 소스 선별 및 정보 수집 |
| researcher | youtube-search | youtube_search.py로 Top5 영상 선별 및 자막 추출 |
| writer | draft-writer | 리서치 기반으로 독자·톤·분량에 맞는 초안 생성 |
| writer | seo-optimizer | 제목·메타설명·키워드 배치·헤딩 구조 SEO 최적화 |

#### 에이전트 간 의존성

```
blog-post 스킬
  └→ researcher (web-search + youtube-search 병렬)
  └→ writer (draft-writer)
  └→ reviewer
  └→ writer (seo-optimizer)
  └→ executor (이미지 병렬 생성)
  └→ generate-docx 스킬 (docx 빌드)
```

---

#### 스킬 목록

| 스킬명 | 유형 | user-invocable | 설명 | 워크플로우 |
|--------|------|:--------------:|------|-----------|
| setup | Setup | true | 의존성 설치 및 API 키 설정 | gateway/install.yaml 기반 설치 |
| help | utility | true | 사용 안내 | 명령 목록 즉시 출력 |
| blog-post | orchestrator | true | 블로그 글 전체 작성 파이프라인 | 옵션수집→리서치→초안→리뷰→SEO→이미지→docx |
| generate-docx | orchestrator | false | docx 빌드 (blog-post에서 위임) | final.md + images → docx 빌드·검증 |

---

#### 스킬 워크플로우

**blog-post 스킬 흐름:**
```
Phase 1: 옵션 수집 (AskUserQuestion)
Phase 2: 정보검색 → Agent: researcher (web-search + youtube-search 병렬)
Phase 3: 초안 작성 → Agent: writer (sub_role=draft-writer)
Phase 4: 리뷰 → Agent: reviewer
Phase 5: SEO 최적화 → Agent: writer (sub_role=seo-optimizer)
Phase 6: 이미지 생성 → Agent: executor
Phase 7: docx 빌드 → Skill: generate-docx
Phase 8: 완료 보고
```

**generate-docx 스킬 흐름 (docx 1단계 패턴):**
```
Phase 1: 입력 데이터 수집 (final.md + images/ 경로)
Phase 2: docx-build-guide 로드
Phase 3: build script 작성 (python-docx 기반)
Phase 4: Build 실행 (Bash)
Phase 5: 파일 존재·크기 검증 (실패 시 재시도 ≤3)
Phase 6: 사용자 보고
```

---

### Gateway 설정 설계

#### install.yaml (설치 매니페스트)

```yaml
custom_tools:
  - name: youtube_search
    description: "YouTube Data API v3 기반 동영상 검색 + 자막 추출"
    source: tools/youtube-search/youtube_search.py
    required: true

  - name: generate_image
    description: "Gemini Nano Banana 모델 기반 이미지 생성"
    source: tools/general/generate_image.py
    required: true

runtime_dependencies:
  - name: python-docx
    description: "DOCX 빌드용 Python 라이브러리"
    runtime: python
    install: "pip install python-docx"
    check: "python -c \"import docx\""
    required: true

  - name: youtube-search-deps
    description: "youtube-search 의존 라이브러리"
    runtime: python
    install: "pip install -r gateway/tools/youtube-search/requirements.txt"
    check: "python -c \"import googleapiclient\""
    required: true

  - name: generate-image-deps
    description: "generate_image 의존 라이브러리"
    runtime: python
    install: "pip install python-dotenv google-genai"
    check: "python -c \"import google.genai\""
    required: true
```

#### runtime-mapping.yaml (추상 → 구체 매핑)

```yaml
tier_mapping:
  HEAVY:
    claude-code: claude-opus-4-7
    cursor: claude-opus-4-7
    codex: gpt-5.4
    antigravity: claude-opus-4-7
  HIGH:
    claude-code: claude-opus-4-7
    cursor: claude-opus-4-7
    codex: gpt-5.4
    antigravity: claude-opus-4-7
  MEDIUM:
    claude-code: claude-sonnet-4-6
    cursor: claude-sonnet-4-6
    codex: gpt-5.4-mini
    antigravity: claude-sonnet-4-6
  LOW:
    claude-code: claude-haiku-4-5-20251001
    cursor: claude-haiku-4-5-20251001
    codex: gpt-5.4-mini
    antigravity: claude-haiku-4-5-20251001

tool_mapping:
  youtube_search:
    - type: custom
      source: "tools/youtube-search/youtube_search.py"
      tools: ["youtube_search"]
  generate_image:
    - type: custom
      source: "tools/general/generate_image.py"
      tools: ["generate_image"]

action_mapping:
  file_write: ["Write", "Edit"]
  file_delete: ["Bash"]
  code_execute: ["Bash"]
  network_access: ["WebFetch", "WebSearch"]
  user_interact: ["AskUserQuestion"]
  agent_delegate: ["Agent"]
```

---

### 디렉토리 구조 설계

```
blog-poster/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
│
├── skills/
│   ├── setup/
│   │   └── SKILL.md
│   ├── help/
│   │   └── SKILL.md
│   ├── blog-post/
│   │   └── SKILL.md
│   └── generate-docx/
│       ├── SKILL.md
│       ├── references/
│       │   ├── docx-build-guide.md      ← 공유자원 복사
│       │   └── docx-build-sample.py     ← 공유자원 복사
│       └── assets/
│           └── docx-builder-SKILL.md    ← 공유자원 복사 (참고용)
│
├── agents/
│   ├── researcher/
│   │   ├── AGENT.md
│   │   ├── agentcard.yaml
│   │   └── tools.yaml
│   ├── writer/
│   │   ├── AGENT.md
│   │   ├── agentcard.yaml
│   │   └── tools.yaml
│   ├── reviewer/
│   │   ├── AGENT.md
│   │   ├── agentcard.yaml
│   │   └── tools.yaml
│   └── executor/
│       ├── AGENT.md
│       ├── agentcard.yaml
│       └── tools.yaml
│
├── .claude/agents/          ← Step 4-A 자동 생성 (Git 커밋 대상)
│   ├── researcher.md
│   ├── writer.md
│   ├── reviewer.md
│   └── executor.md
├── .cursor/agents/
│   └── (동일 4개)
├── .codex/agents/
│   └── (동일 4개, .toml)
├── .antigravity/agents/
│   └── (동일 4개)
│
├── gateway/
│   ├── install.yaml
│   ├── runtime-mapping.yaml
│   └── tools/
│       ├── .env                        ← YOUTUBE_API_KEY, GEMINI_API_KEY
│       ├── youtube-search/             ← 공유자원 복사
│       │   ├── youtube_search.py
│       │   └── requirements.txt
│       └── general/                    ← 공유자원 복사
│           └── generate_image.py
│
├── commands/
│   ├── setup.md
│   ├── help.md
│   └── blog-post.md
│
├── output/                             ← 산출물 디렉토리
│   └── {주제}/
│       ├── 0.research.md
│       ├── 1.draft.md
│       ├── 2.revise.md
│       ├── 3.seo-optimize.md
│       ├── 4.final.md
│       ├── 5.{주제}.docx
│       └── images/
│
├── AGENTS.md
├── CLAUDE.md
├── .gitignore
└── README.md
```

---

## 개발 계획

### 3.1 개발 순서 (순차적)

| 순번 | 단계 | 파일/디렉토리 | 검증 방법 |
|------|------|--------------|----------|
| 1 | 스켈레톤 생성 | `.claude-plugin/`, `.gitignore` | 디렉토리 존재 확인 |
| 2 | 매니페스트 작성 | `.claude-plugin/plugin.json`, `marketplace.json` | JSON 구조 확인 |
| 3 | Gateway 설정 | `gateway/install.yaml`, `runtime-mapping.yaml` | YAML 파싱 확인 |
| 4 | 공유자원 복사 | `gateway/tools/`, `skills/generate-docx/references/` | 파일 존재 확인 |
| 5 | agents 개발 (병렬 가능) | `agents/researcher/`, `writer/`, `reviewer/`, `executor/` | 표준 준수 체크리스트 |
| 6 | 런타임 어댑터 생성 (Step 4-A) | `.claude/`, `.cursor/`, `.codex/`, `.antigravity/` | 4런타임 × 4에이전트 = 16파일 |
| 7 | skills/generate-docx | `skills/generate-docx/SKILL.md` | docx 1단계 패턴 포함 |
| 8 | skills/blog-post | `skills/blog-post/SKILL.md` | 에이전트 호출 규칙 포함 |
| 9 | skills/setup | `skills/setup/SKILL.md` | install.yaml 참조 확인 |
| 10 | skills/help | `skills/help/SKILL.md` | 즉시 출력 방식 확인 |
| 11 | commands/ 진입점 | `commands/setup.md`, `help.md`, `blog-post.md` | 슬래시 명령 형식 확인 |
| 12 | Phase 4: AGENTS.md, CLAUDE.md | `AGENTS.md`, `CLAUDE.md` | 팀 구성·변수 확인 |
| 13 | README.md | `README.md` | 필수 6섹션 확인 |

### 3.2 병렬 가능 단계

- 순번 5: researcher / writer / reviewer / executor 에이전트 4개 병렬 개발

### 3.3 공유 자원 활용 계획

| 자원명 | 활용 위치 | 활용 방법 |
|--------|----------|----------|
| youtube-search | gateway/tools/ + executor tools.yaml | Bash로 직접 실행 |
| generate_image | gateway/tools/ + executor tools.yaml | Bash로 직접 실행 |
| docx-build-guide | generate-docx/references/ | SKILL.md에서 가이드 로드 후 빌드 스크립트 작성 |
| docx-builder-SKILL | generate-docx/assets/ | SKILL.md 초안 참고 템플릿 |
| docx-build-sample | generate-docx/references/ | 빌드 스크립트 작성 시 샘플 참조 |

### 3.4 기술 요구사항 확인

#### Python 라이브러리
| 라이브러리 | 버전 | 용도 |
|-----------|------|------|
| python-docx | 최신 | docx 빌드 |
| google-api-python-client | 최신 | YouTube Data API |
| youtube-transcript-api | ≥1.0.0 | YouTube 자막 추출 |
| python-dotenv | 최신 | 환경변수 로드 |
| google-genai | 최신 | Gemini 이미지 생성 |

#### 환경 변수 (`.env`)
```
YOUTUBE_API_KEY=<YouTube Data API v3 키>
GEMINI_API_KEY=<Google Gemini API 키>
```
파일 위치: `gateway/tools/.env`

---

## MS-Office 산출물 패턴 (docx 1단계 패턴 적용)

감지 형식: **docx**
패턴: 빌더 스킬 단독 1단계 패턴 (spec-writer 에이전트 미생성)

적용 방식:
- `skills/generate-docx/SKILL.md` 에 Build & Verify Phase 포함
- `gateway/install.yaml`의 `runtime_dependencies`에 python-docx 등록
- `setup` 스킬이 python-docx 설치 처리
- 외부 변환 스킬(anthropic-skills:docx 등) 호출 금지
