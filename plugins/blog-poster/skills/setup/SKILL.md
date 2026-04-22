---
name: setup
description: blog-poster 플러그인 초기 설정 (Python 의존성 설치, API Key 등록, 모델 매핑 확인)
type: setup
user-invocable: true
---

# Setup

[SETUP 활성화]

## 목표

blog-poster 플러그인의 Python 의존성 라이브러리를 설치하고,
YouTube API Key와 Gemini API Key를 `gateway/tools/.env`에 저장하며,
`runtime-mapping.yaml`의 모델 버전을 사용자에게 확인받아 최신 상태로 유지한다.

## 활성화 조건

사용자가 `/blog-poster:setup` 호출 시 또는 "blog-poster 설치", "blog-poster 설정" 키워드 감지 시.

## 워크플로우

### Phase 1: 환경 전제 확인

Python ≥ 3.10 설치 여부 확인:
```bash
python --version
```
없으면 사용자에게 설치 안내 후 중단.

### Phase 2: gateway/install.yaml 로드 및 파싱

`gateway/install.yaml`을 읽어 `runtime_dependencies`와 `custom_tools` 항목 추출.

### Phase 3: Python 의존성 설치

각 `runtime_dependencies` 항목에 대해:
1. `check` 명령으로 이미 설치 여부 확인
2. 미설치 시 `install` 명령 실행
3. `required: true` 항목 실패 시 중단 및 안내

```bash
# youtube-search 의존성
python -c "import googleapiclient" 2>/dev/null || pip install -r gateway/tools/youtube-search/requirements.txt

# generate_image 의존성
python -c "import google.genai" 2>/dev/null || pip install python-dotenv google-genai

# python-docx
python -c "import docx" 2>/dev/null || pip install python-docx
```

### Phase 4: 커스텀 도구 확인

`custom_tools` 항목의 `source` 파일 존재 확인:
- `gateway/tools/youtube-search/youtube_search.py`
- `gateway/tools/general/generate_image.py`

없으면 공유자원 복사 안내.

### Phase 5: API Key 설정

`gateway/tools/.env` 파일에 API Key 설정:
1. 파일이 없으면 생성
2. YOUTUBE_API_KEY 값이 비어있으면 사용자에게 입력 요청
3. GEMINI_API_KEY 값이 비어있으면 사용자에게 입력 요청
4. 입력받은 값을 .env 파일에 저장

### Phase 6: 모델 매핑 확인

`gateway/runtime-mapping.yaml`의 현재 모델 매핑을 사용자에게 보여주고 확인 요청.
최신 Claude 모델로 업데이트할지 묻고, 갱신 확정 시:
- `runtime-mapping.yaml` 업데이트
- `.claude/agents/`, `.cursor/agents/`, `.codex/agents/`, `.antigravity/agents/`의
  모든 스텁 파일의 frontmatter `model:` 필드 동기화

### Phase 7: 설치 결과 보고

설치된 항목 목록과 설정 완료 항목을 보고한다.

## 문제 해결

| 증상 | 해결 |
|------|------|
| `pip install` 실패 | Python 3.10+ 확인, `pip install -U pip` 후 재시도 |
| YOUTUBE_API_KEY 오류 | Google Cloud Console에서 YouTube Data API v3 활성화 확인 |
| GEMINI_API_KEY 오류 | Google AI Studio에서 API Key 발급 확인 |
| youtube_search.py 실행 오류 | `python gateway/tools/youtube-search/youtube_search.py --help` 확인 |
| generate_image.py 실행 오류 | `python gateway/tools/general/generate_image.py --help` 확인 |
