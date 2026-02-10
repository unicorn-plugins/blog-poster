---
name: write-post
description: 블로그 글 작성 메인 워크플로우 (6단계 순차 진행)
user-invocable: true
---

# Write Post

[BLOG-POSTER WRITE-POST 활성화]

---

## 목표

사용자가 입력한 블로그 주제를 기반으로 리서치 → 초안 작성 → SEO 최적화 → 이미지 생성 → 최종 Word 문서 출력까지
6단계 워크플로우를 순차적으로 오케스트레이션함.

[Top](#write-post)

---

## 활성화 조건

사용자가 `/blog-poster:write-post` 호출 시 또는
"블로그 써줘", "글 작성해줘", "포스트 만들어" 키워드 감지 시.

[Top](#write-post)

---

## 에이전트 호출 규칙

### 에이전트 FQN

| 에이전트 | FQN |
|----------|-----|
| researcher | `blog-poster:researcher:researcher` |
| writer | `blog-poster:writer:writer` |
| seo-optimizer | `blog-poster:seo-optimizer:seo-optimizer` |
| image-creator | `blog-poster:image-creator:image-creator` |

### 프롬프트 조립

1. `agents/{agent-name}/` 에서 3파일 로드 (AGENT.md + agentcard.yaml + tools.yaml)
2. `gateway/runtime-mapping.yaml` 참조하여 구체화:
   - **모델 구체화**: agentcard.yaml의 `tier` → `tier_mapping`에서 모델 결정
   - **툴 구체화**: tools.yaml의 추상 도구 → `tool_mapping`에서 실제 도구 결정
   - **금지액션 구체화**: agentcard.yaml의 `forbidden_actions` → `action_mapping`에서 제외할 실제 도구 결정
   - **최종 도구** = (구체화된 도구) - (제외 도구)
3. 프롬프트 조립: AGENT.md + agentcard.yaml + tools.yaml을 합쳐 하나의 프롬프트로 구성
   - **구성 순서**: 공통 정적(runtime-mapping) → 에이전트별 정적(3파일) → 동적(작업 지시)
4. `Task(subagent_type=FQN, model=구체화된 모델, prompt=조립된 프롬프트)` 호출

[Top](#write-post)

---

## 워크플로우

### Phase 1: 주제 입력

사용자에게 블로그 주제를 확인함.

- 사용자 메시지에 주제가 포함된 경우 → 바로 Phase 2 진행
- 주제가 불분명한 경우 → AskUserQuestion으로 주제 입력 요청
- 추가 확인: 특별히 원하는 톤/스타일, 대상 독자가 있는지 확인 (없으면 기본값 사용)

기본값:
- 톤: 친근하고 캐주얼한 문체
- 독자: 기술/IT에 관심 있는 일반인
- 분량: 1,000~2,000자

### Phase 2: 리서치 → Agent: researcher (`/oh-my-claudecode:research` 활용)

- **TASK**: 주제 "{사용자 입력 주제}"에 대해 웹 검색, 기술 문서 조사, 트렌드 분석 수행
- **EXPECTED OUTCOME**: 구조화된 리서치 보고서 (트렌드 요약, 핵심 자료, 참고 링크, 활용 포인트)
- **MUST DO**: 최소 3개 이상 서로 다른 출처에서 자료 수집, 모든 자료에 출처 명시
- **MUST NOT DO**: 직접 글을 작성하지 않음, 출처 없는 정보를 포함하지 않음
- **CONTEXT**: 대상 독자는 기술/IT 관심 일반인, 한국어 블로그용 자료 수집

### Phase 3: 초안 작성 → Agent: writer (`ulw` 활용)

- **TASK**: Phase 2의 리서치 결과를 기반으로 한국어 블로그 글 초안 작성 (1,000~2,000자)
- **EXPECTED OUTCOME**: 제목, 소제목, 인트로, 본론, 결론이 포함된 블로그 글 초안 (Markdown 형식)
- **MUST DO**: 친근하고 캐주얼한 톤("~해요" 체), 리서치 출처 반영, 구조화된 소제목 사용
- **MUST NOT DO**: 2,000자를 초과하지 않음, 딱딱한 논문 스타일 사용 금지
- **CONTEXT**: Phase 2 리서치 보고서, 대상 독자 정보, 톤/스타일 가이드

### Phase 4: SEO 최적화 → Agent: seo-optimizer (`ulw` 활용)

- **TASK**: Phase 3의 블로그 초안에 대해 SEO 분석 및 최적화 제안
- **EXPECTED OUTCOME**: 키워드 분석, 최적화된 제목 후보 2~3개, 메타 태그, SEO 점수 보고서
- **MUST DO**: 키워드 밀도 평가, 메타 description 150~160자, 제목 50~60자
- **MUST NOT DO**: 직접 글을 수정하지 않음 (최적화 제안만 반환)
- **CONTEXT**: Phase 3 블로그 초안, 주제 키워드

### Phase 5: 이미지 생성 → Agent: image-creator (`ulw` 활용)

- **TASK**: 블로그 주제에 맞는 삽화 또는 썸네일 이미지 1장 생성
- **EXPECTED OUTCOME**: PNG 이미지 파일 1개 + 이미지 배치 제안
- **MUST DO**: 블로그 주제와 관련된 프롬프트 작성, 흰색 배경 스타일
- **MUST NOT DO**: 글을 수정하지 않음
- **CONTEXT**: Phase 3 블로그 초안 내용, 주제 키워드

> **참고**: GEMINI_API_KEY가 설정되지 않은 경우 이 Phase를 건너뛰고 Phase 6으로 진행.
> 사용자에게 이미지 생성을 건너뛰었음을 안내.

### Phase 6: 최종 문서 출력 → Agent: writer (`ulw` 활용)

- **TASK**: SEO 최적화 결과와 이미지를 반영하여 최종 Word(.docx) 문서 생성
- **EXPECTED OUTCOME**: 완성된 Word(.docx) 파일 (제목, 메타 정보, 본문, 이미지 포함)
- **MUST DO**: python-docx로 .docx 생성, SEO 제목 반영, 이미지 삽입 (있는 경우), 참고 자료 포함
- **MUST NOT DO**: 글 톤/스타일을 변경하지 않음
- **CONTEXT**: Phase 3 초안 + Phase 4 SEO 결과 + Phase 5 이미지 파일 경로

[Top](#write-post)

---

## 완료 조건

- [ ] 리서치 보고서가 생성되었음
- [ ] 블로그 초안이 1,000~2,000자로 작성되었음
- [ ] SEO 최적화 보고서가 생성되었음
- [ ] 이미지가 생성되었음 (또는 GEMINI_API_KEY 미설정으로 건너뜀)
- [ ] Word(.docx) 파일이 정상적으로 생성되었음

[Top](#write-post)

---

## 검증 프로토콜

각 Phase 완료 후 결과물을 확인:

1. **Phase 2 검증**: 리서치 보고서에 3개 이상 출처가 포함되었는지 확인
2. **Phase 3 검증**: 글 분량 1,000~2,000자, 한국어, 캐주얼 톤 확인
3. **Phase 4 검증**: SEO 점수와 최적화 제안이 포함되었는지 확인
4. **Phase 5 검증**: 이미지 파일이 생성되었는지 확인 (PNG)
5. **Phase 6 검증**: .docx 파일이 정상 생성되었는지 확인

검증 실패 시 해당 Phase를 재실행.

[Top](#write-post)

---

## 상태 정리

워크플로우 완료 시:
- 중간 산출물(리서치 보고서, 초안, SEO 보고서)은 output/ 디렉토리에 보존
- 최종 Word 파일은 사용자에게 전달

[Top](#write-post)

---

## 취소

사용자가 "취소", "중단", "그만" 키워드 입력 시 즉시 워크플로우 중단.
현재까지의 진행 상황을 보고하고 중간 산출물을 보존함.

[Top](#write-post)

---

## 재개

이전 워크플로우의 중간 산출물(output/ 디렉토리)이 있으면
마지막 완료된 Phase부터 재시작 가능.

[Top](#write-post)

---

## 출력 형식

워크플로우 완료 시 사용자에게 보고:

```
## 블로그 글 작성 완료!

📝 제목: {최종 제목}
📊 SEO 점수: {점수}/100
🖼️ 이미지: {생성 여부}
📄 문서: {파일 경로}

### 생성된 파일
- 최종 문서: output/{주제}-blog.docx
- 리서치 보고서: output/{주제}-research.md
- SEO 보고서: output/{주제}-seo.md
- 이미지: output/{주제}-image.png
```

[Top](#write-post)

---

## MUST 규칙

| # | 규칙 |
|---|------|
| 1 | 6단계 워크플로우를 순차적으로 수행 (Phase 순서 준수) |
| 2 | 각 Phase의 결과물을 다음 Phase에 전달 |
| 3 | 에이전트 호출 시 FQN, 프롬프트 조립, 모델 구체화 규칙 준수 |
| 4 | GEMINI_API_KEY 미설정 시 Phase 5를 건너뛰고 사용자에게 안내 |
| 5 | 최종 문서는 반드시 Word(.docx) 형식으로 출력 |
| 6 | 각 Phase 완료 시 사용자에게 진행 상황 보고 |

[Top](#write-post)

---

## MUST NOT 규칙

| # | 금지 사항 |
|---|----------|
| 1 | Phase 순서를 건너뛰거나 역순으로 실행하지 않음 |
| 2 | 오케스트레이터가 직접 글을 작성하거나 리서치하지 않음 (에이전트에 위임) |
| 3 | 에이전트의 내부 사고 방식이나 단계별 절차를 지시하지 않음 |
| 4 | 사용자 확인 없이 주제를 임의로 결정하지 않음 |

[Top](#write-post)

---

## 검증 체크리스트

- [ ] 6단계 워크플로우가 순차적으로 정의되어 있는가
- [ ] 모든 Phase에 `→ Agent:` 마커와 5항목이 포함되어 있는가
- [ ] 에이전트 호출 규칙(FQN, 프롬프트 조립)이 포함되어 있는가
- [ ] 완료 조건, 검증 프로토콜, 상태 정리, 취소/재개 섹션이 있는가
- [ ] GEMINI_API_KEY 미설정 시 대응이 정의되어 있는가
- [ ] 프롬프트 구성 순서가 공통 정적 → 에이전트별 정적 → 동적 순서인가
- [ ] 모든 워크플로우 단계에 오케스트레이션 스킬 활용이 명시되었는가

[Top](#write-post)
