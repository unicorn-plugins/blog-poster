---
name: generate-docx
description: final.md와 생성된 이미지를 기반으로 docx 문서를 빌드하고 검증
type: orchestrator
user-invocable: false
---

# Generate DOCX

## 목표

`output/{주제}/4.final.md`와 `output/{주제}/images/`의 이미지를 기반으로
python-docx를 사용하여 docx 문서를 빌드하고 검증한다.
이미지는 docx 본문에 임베드된다.

## 활성화 조건

blog-post 스킬의 Phase 7에서 위임될 때 활성화됨.

## 에이전트 호출 규칙

이 스킬은 에이전트를 호출하지 않음. 직결형 패턴으로 직접 Build & Verify를 수행함.

## 워크플로우

### Phase 1: 입력 데이터 수집

`output/{주제}/4.final.md`를 Read 도구로 읽는다.
`output/{주제}/images/` 디렉토리의 이미지 파일 목록을 확인한다.
`skills/generate-docx/references/docx-build-guide.md`를 읽어 빌드 규칙을 파악한다.

### Phase 2: Build script 작성

`skills/generate-docx/references/docx-build-sample.py`를 참조하여
아래 요구사항을 충족하는 `output/{주제}/build_docx.py`를 Write 도구로 작성한다:

**빌드 요구사항:**
- python-docx 사용
- 한글 폰트: 맑은 고딕 (제목), 나눔고딕 또는 맑은 고딕 (본문)
- H1 → 제목 스타일, H2 → 제목1, H3 → 제목2
- 일반 텍스트 → Normal 스타일, 글꼴 11pt
- 이미지 삽입: final.md의 `![alt](images/파일명)` 패턴 감지 → docx에 이미지 임베드
- 이미지 너비: 최대 15cm (비율 유지)
- 출력 파일: `output/{주제}/5.{주제}.docx`

### Phase 3: Build 실행

Bash 도구로 빌드 스크립트를 실행한다:
```bash
python output/{주제}/build_docx.py
```

### Phase 4: 검증

다음 항목을 검증한다:
1. `output/{주제}/5.{주제}.docx` 파일이 존재하는지 확인
2. 파일 크기가 0보다 큰지 확인
3. 실패 시 오류 분석 후 스크립트 수정 → 재빌드 (최대 3회)

### Phase 5: 사용자 보고

완료 후 다음을 보고한다:
- 생성된 docx 절대 경로
- 파일 크기
- 삽입된 이미지 수
- 빌드 스크립트 경로

## 완료 조건

- [ ] `output/{주제}/5.{주제}.docx` 파일 존재 및 크기 > 0
- [ ] 이미지가 docx에 정상 임베드됨

## 상태 정리

완료 후 임시 빌드 스크립트(`output/{주제}/build_docx.py`)는 유지함 (재빌드 참조용).
