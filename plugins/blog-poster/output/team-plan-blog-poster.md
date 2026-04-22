# 팀 기획서

## 기본 정보
- 플러그인명: blog-poster
- 목표: 특정 주제에 대한 정보를 검색하여 블로그 글을 작성
- 대상 도메인: 콘텐츠 제작
- 대상 사용자: 기획자

## 핵심기능
- 정보검색: 웹, 유튜브에서 사용자가 지정한 AI 주제를 검색
- 블로그글 작성: 사용자가 선택한 주제, 블로그 독자, 글의 톤앤매너, 글의 분량에 맞게 블로그 글 작성

## 사용자 플로우
### 옵션 수집
- 글 주제, 블로그 독자, 글의 톤앤매너, 글의 분량을 AskUserQuestion으로 수집

### 정보검색
- Step 1. 웹/유튜브 검색 (병렬 수행)
    - 공신력 있는 웹사이트를 선별하여 사용자가 지정한 AI 주제를 검색
    - 유튜브에서 사용자가 지정한 AI 주제의 영상을 검색하고 조회수가 높은 Top5 선별
- Step 2. 콘텐츠 종합
  - 정보 검색 결과를 마크다운 파일로 저장: `output/{주제}/0.research.md`

### 블로그글 작성
- Step 1. 초안 작성 → `output/{주제}/1.draft.md`
- Step 2. 리뷰/업데이트: 리뷰 및 반영 → `output/{주제}/2.revise.md`
- Step 3. SEO 최적화 → `output/{주제}/3.seo-optimize.md`
- Step 4. 이미지 생성: 글의 내용 이해를 돕기 위한 이미지 생성. 이미지 생성 대상을 먼저 선별하고 이미지 생성은 병렬로 수행 → `output/{주제}/4.final.md`
- Step 5. 문서 작성 → `output/{주제}/5.{주제}.docx`

## 에이전트 구성
- researcher: 웹/유튜브 병렬 검색 및 콘텐츠 종합 전담 (MEDIUM / Sonnet)
  - 웹검색: 공신력 있는 소스를 선별하여 주제 관련 최신 정보 수집
  - 유튜브검색: youtube-search 도구로 주제 관련 Top5 영상 선별 및 자막 추출
- writer: 블로그 초안 작성 및 SEO 최적화 전담 (MEDIUM / Sonnet)
  - 초안작성: 수집 정보 기반으로 독자·톤·분량에 맞는 블로그 초안 생성
  - seo-optimizer: 제목·메타설명·키워드 배치 등 SEO 최적화 적용
- reviewer: 초안 품질 검토 및 개선 의견 도출 (HIGH / Opus)
- executor: 이미지 생성(병렬) 및 docx 빌드 실행 (LOW~MEDIUM / Haiku~Sonnet)

## 공유자원
| 자원 유형 | 자원명 | 자원 경로 |
|----------|--------|-----------|
| 도구 | youtube-search | C:/Users/hiond/plugins/dmap/resources/tools/customs/youtube-search/youtube_search.py |
| 도구 | generate_image | C:/Users/hiond/plugins/dmap/resources/tools/customs/general/generate_image.py |
| 가이드 | docx-build-guide | C:/Users/hiond/plugins/dmap/resources/guides/office/docx-build-guide.md |
| 템플릿 | docx-builder-SKILL | C:/Users/hiond/plugins/dmap/resources/templates/office/docx-builder-SKILL.md |
| 샘플 | docx-build-sample | C:/Users/hiond/plugins/dmap/resources/samples/office/docx-build-sample.py |
