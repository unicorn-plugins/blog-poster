---
name: blog-post
description: 주제 검색부터 docx 문서 산출까지 블로그 글 전체 작성 파이프라인
type: orchestrator
user-invocable: true
---

# Blog Post

[BLOG-POST 활성화]

## 목표

사용자가 지정한 주제에 대해 웹/유튜브 리서치 → 초안 작성 → 리뷰 → SEO 최적화 →
이미지 생성 → docx 빌드까지 전체 파이프라인을 오케스트레이션한다.

## 활성화 조건

사용자가 `/blog-poster:blog-post` 호출 시 또는 "블로그 글 작성", "블로그 포스팅", "AI 주제 블로그" 키워드 감지 시.

## 에이전트 호출 규칙

### 에이전트 FQN

| 에이전트 | FQN |
|---------|-----|
| researcher | `blog-poster:researcher:researcher` |
| writer | `blog-poster:writer:writer` |
| reviewer | `blog-poster:reviewer:reviewer` |
| executor | `blog-poster:executor:executor` |

### 프롬프트 조립

각 에이전트 호출 시:
1. `agents/{agent-name}/AGENT.md` 읽기
2. `agents/{agent-name}/agentcard.yaml` 읽기
3. `agents/{agent-name}/tools.yaml` 읽기 (있는 경우)
4. `gateway/runtime-mapping.yaml`에서 tier→모델 매핑
5. `Agent(subagent_type=FQN, model=매핑된_모델, prompt=AGENT.md+agentcard.yaml+tools.yaml+작업지시)` 호출

tier → model 매핑 (MEDIUM): `gateway/runtime-mapping.yaml`의 `tier_mapping.MEDIUM.claude-code` 참조

### 서브 에이젼트 호출
워크플로우 단계에 `Agent: {agent-name}`이 명시된 경우, 해당 에이전트를 위 프롬프트 조립 규칙에 따라 호출

## 워크플로우

### Phase 1: 옵션 수집

AskUserQuestion으로 다음 항목을 수집한다:
- 글 주제 (예: "Claude AI 활용법")
- 블로그 독자 (예: "IT 비전공 직장인")
- 글의 톤앤매너 (예: "친근하고 쉽게", "전문적이고 간결하게")
- 글의 분량 (예: "2000자", "3000자 이상")

수집 후 `output/{주제}/` 디렉토리를 생성한다.

### Phase 2: 정보검색 → Agent: researcher (web-search + youtube-search 병렬)

- **TASK**: 주제에 대해 웹과 유튜브를 병렬로 검색하여 `output/{주제}/0.research.md`에 종합 저장
- **EXPECTED OUTCOME**: `output/{주제}/0.research.md` — 웹 5개 이상 소스 + 유튜브 Top5 영상 요약
- **MUST DO**: 두 세부역할(web-search, youtube-search)을 병렬로 수행할 것
- **MUST NOT DO**: 블로그 초안 작성 금지, 리서치 범위 외 작업 금지
- **CONTEXT**: 주제={주제}, 독자={독자}, 톤={톤앤매너}, 분량={분량}

### Phase 3: 초안 작성 → Agent: writer (sub_role=draft-writer)

- **TASK**: 리서치 결과를 기반으로 한국어 블로그 초안 작성 후 `output/{주제}/1.draft.md` 저장
- **EXPECTED OUTCOME**: `output/{주제}/1.draft.md` — 지정 분량(±10%) 한국어 초안
- **MUST DO**: `output/{주제}/0.research.md` 참조, 독자·톤·분량 기준 준수
- **MUST NOT DO**: SEO 최적화 금지, 이미지 생성 금지
- **CONTEXT**: 독자={독자}, 톤앤매너={톤앤매너}, 분량={분량}, 리서치 파일=`output/{주제}/0.research.md`

### Phase 4: 리뷰 → Agent: reviewer

- **TASK**: 초안을 가독성·정확성·구성 관점에서 검토하고 수정본을 `output/{주제}/2.revise.md`에 저장
- **EXPECTED OUTCOME**: `output/{주제}/2.revise.md` — 개선 의견 + 수정된 본문
- **MUST DO**: `output/{주제}/0.research.md`와 `output/{주제}/1.draft.md` 모두 참조하여 사실 확인
- **MUST NOT DO**: SEO 최적화 금지, 원본 의도 훼손 금지
- **CONTEXT**: 초안 파일=`output/{주제}/1.draft.md`, 리서치 파일=`output/{주제}/0.research.md`

### Phase 5: SEO 최적화 → Agent: writer (sub_role=seo-optimizer)

- **TASK**: 수정본에 SEO 최적화 적용 후 `output/{주제}/3.seo-optimize.md` 저장
- **EXPECTED OUTCOME**: `output/{주제}/3.seo-optimize.md` — 제목·메타·키워드·헤딩 구조 최적화
- **MUST DO**: `output/{주제}/2.revise.md` 기반, 주제 키워드 자연스럽게 배치
- **MUST NOT DO**: 본문 내용 임의 변경 금지, 키워드 과도 삽입 금지
- **CONTEXT**: 수정본 파일=`output/{주제}/2.revise.md`, 주제={주제}

### Phase 6: 이미지 생성 → Agent: executor

- **TASK**: SEO 파일 분석 후 이미지 필요 위치 선별, generate_image.py로 병렬 생성, final.md 저장
- **EXPECTED OUTCOME**: `output/{주제}/images/image_001.png` ~ `image_00N.png` + `output/{주제}/4.final.md`
- **MUST DO**: generate_image.py 사용, 병렬 생성, final.md에 이미지 삽입 위치 명시
- **MUST NOT DO**: 이미지 5개 초과 금지, 본문 내용 수정 금지
- **CONTEXT**: SEO 파일=`output/{주제}/3.seo-optimize.md`, 이미지 저장 경로=`output/{주제}/images/`, generate_image 위치=`gateway/tools/general/generate_image.py`

### Phase 7: docx 빌드 → Skill: generate-docx

- **INTENT**: final.md + 이미지를 기반으로 docx 문서를 빌드하여 최종 산출물 생성
- **ARGS**: 주제={주제}, final.md 경로=`output/{주제}/4.final.md`, 이미지 디렉토리=`output/{주제}/images/`
- **RETURN**: `output/{주제}/5.{주제}.docx` 파일 생성 완료

### Phase 8: 완료 보고

다음을 사용자에게 보고한다:
- 전체 산출물 목록 (0.research.md ~ 5.{주제}.docx)
- 각 파일의 절대 경로
- 블로그 글 분량 및 이미지 수
- 소요 시간 요약

## 완료 조건

- [ ] `output/{주제}/0.research.md` 존재
- [ ] `output/{주제}/1.draft.md` 존재
- [ ] `output/{주제}/2.revise.md` 존재
- [ ] `output/{주제}/3.seo-optimize.md` 존재
- [ ] `output/{주제}/4.final.md` 존재 + 이미지 참조 포함
- [ ] `output/{주제}/images/` 이미지 파일 1개 이상 존재
- [ ] `output/{주제}/5.{주제}.docx` 존재 + 크기 > 0

## 상태 정리

진행 상태를 `output/{주제}/.state.json`에 기록:
```json
{
  "topic": "{주제}",
  "phases_completed": ["phase-1", "phase-2", ...],
  "current_phase": "phase-N"
}
```
완료 시 상태 파일 삭제.

## 재개

상태 파일 존재 시 마지막 완료 단계 다음부터 재시작.
`output/{주제}/` 디렉토리에서 존재하는 파일을 확인하여 완료된 단계 판단.
