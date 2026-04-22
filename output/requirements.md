# 요구사항 정의서

## 기본 정보
- 플러그인명: blog-poster
- 목표: 특정 주제에 대한 정보를 검색하여 블로그 글을 작성
- 대상 도메인: 콘텐츠 제작
- 대상 사용자: 기획자
- 작성자: 이해경

---

## 핵심기능

### 정보검색
- Claude Code 내장 WebSearch 도구로 공신력 있는 웹사이트(뉴스, 학술, 공식 문서 등)를 선별하여 AI 주제 검색
- youtube-search 도구(YouTube Data API v3)로 주제 관련 영상 검색 → 조회수 기준 Top5 선별 및 자막 추출
- 웹/유튜브 검색은 병렬 수행
- 수집 결과를 마크다운으로 정리: `output/{주제}/0.research.md`

### 블로그글 작성
- 작성 언어: 한국어
- 사용자가 지정한 주제, 독자, 톤앤매너, 분량을 반영하여 블로그 글 작성
- 초안 → 리뷰/수정 → SEO 최적화 → 이미지 생성 → docx 빌드 순서로 진행
- 이미지는 generate_image 도구(Gemini Nano Banana 모델)로 생성하고 docx에 임베드

---

## 사용자 플로우

### 옵션 수집
- AskUserQuestion으로 다음 항목 수집:
  - 글 주제 (예: "Claude AI 활용법")
  - 블로그 독자 (예: "IT 비전공 직장인")
  - 글의 톤앤매너 (예: "친근하고 쉽게", "전문적이고 간결하게")
  - 글의 분량 (예: "2000자", "3000자 이상")

### 정보검색
- Step 1. 웹/유튜브 검색 (병렬 수행)
  - WebSearch: 공신력 있는 소스(뉴스, 공식 문서, 학술 자료) 우선 선별, 최소 5개 출처 수집
  - youtube-search: 주제 관련 영상 검색, 자막 있는 영상 Top5 선별 (최근 1년 이내 우선)
- Step 2. 콘텐츠 종합
  - 수집 정보를 구조화하여 `output/{주제}/0.research.md`에 저장
  - 섹션: 웹 검색 결과 요약 / 유튜브 영상 요약 / 핵심 인사이트

### 블로그글 작성
- Step 1. 초안 작성 → `output/{주제}/1.draft.md`
  - 수집된 리서치 기반으로 독자·톤·분량에 맞는 블로그 초안 생성
- Step 2. 리뷰/업데이트 → `output/{주제}/2.revise.md`
  - 가독성, 정확성, 구성 관점에서 리뷰 후 반영
- Step 3. SEO 최적화 → `output/{주제}/3.seo-optimize.md`
  - 제목 최적화, 메타 설명, 키워드 배치, 헤딩 구조 개선
- Step 4. 이미지 생성 → `output/{주제}/4.final.md`
  - 글 내용 이해를 돕는 이미지 생성 대상 선별 (최대 3~5개)
  - generate_image 도구로 병렬 이미지 생성 → `output/{주제}/images/` 저장
  - 이미지 삽입 위치를 final.md에 명시
- Step 5. docx 빌드 → `output/{주제}/5.{주제}.docx`
  - final.md + 생성된 이미지를 기반으로 docx 빌드
  - 이미지 docx에 임베드

---

## 기술 요구사항

### 도구 스택
| 도구 | 용도 | 비고 |
|------|------|------|
| Claude Code 내장 WebSearch | 웹 검색 | API 키 불필요 |
| youtube-search (커스텀) | YouTube 검색·자막 추출 | YOUTUBE_API_KEY 필요 |
| generate_image (커스텀) | 이미지 생성 | GEMINI_API_KEY 필요 |
| python-docx | docx 빌드 | pip install python-docx |

### 환경 변수
| 변수명 | 용도 | 위치 |
|--------|------|------|
| YOUTUBE_API_KEY | YouTube Data API v3 키 | gateway/tools/.env |
| GEMINI_API_KEY | Gemini 이미지 생성 API 키 | gateway/tools/.env |

### 출력 경로 구조
```
output/{주제}/
├── 0.research.md       # 리서치 결과
├── 1.draft.md          # 초안
├── 2.revise.md         # 리뷰/수정본
├── 3.seo-optimize.md   # SEO 최적화본
├── 4.final.md          # 이미지 삽입 최종본
├── 5.{주제}.docx       # 최종 docx 산출물
└── images/             # 생성된 이미지 파일
    ├── image_001.png
    └── ...
```

---

## 공유자원

| 자원 유형 | 자원명 | 자원 경로 |
|----------|--------|-----------|
| 도구 | youtube-search | C:/Users/hiond/plugins/dmap/resources/tools/customs/youtube-search/youtube_search.py |
| 도구 | generate_image | C:/Users/hiond/plugins/dmap/resources/tools/customs/general/generate_image.py |
| 가이드 | docx-build-guide | C:/Users/hiond/plugins/dmap/resources/guides/office/docx-build-guide.md |
| 템플릿 | docx-builder-SKILL | C:/Users/hiond/plugins/dmap/resources/templates/office/docx-builder-SKILL.md |
| 샘플 | docx-build-sample | C:/Users/hiond/plugins/dmap/resources/samples/office/docx-build-sample.py |
