---
name: setup
description: blog-poster 플러그인 초기 설정
user-invocable: true
---

# Setup

[BLOG-POSTER SETUP 활성화]

---

## 목표

blog-poster 플러그인의 초기 설정을 수행함.
`gateway/install.yaml`을 읽어 필요한 MCP 서버와 커스텀 도구를 설치하고,
플러그인 활성화를 확인함.

[Top](#setup)

---

## 활성화 조건

사용자가 `/blog-poster:setup` 호출 시.

[Top](#setup)

---

## 사용자 상호작용

모든 설정 단계에서 AskUserQuestion 도구를 사용하여 사용자에게 선택지를 제공함.

[Top](#setup)

---

## 워크플로우

### Step 1: 환경 확인

1. Node.js 설치 여부 확인 (`node --version`)
2. Python 3 설치 여부 확인 (`python3 --version`)
3. 미설치 항목이 있으면 사용자에게 안내 후 계속 진행 여부 확인

### Step 2: install.yaml 읽기

1. `gateway/install.yaml` 파일을 읽어 설치 대상 파악
2. 각 항목의 설치 상태 확인

### Step 3: MCP 서버 설치

`install.yaml`의 `mcp_servers` 항목 순회:

1. **context7** (선택):
   - `claude mcp list -s user`로 이미 설치 여부 확인
   - 미설치 시 `gateway/mcp/context7.json` 설정 파일을 읽어 등록:
     ```
     claude mcp add-json context7 '<config-json>' -s user
     ```
   - `required: false`이므로 실패해도 경고만 출력하고 계속 진행

### Step 4: 커스텀 도구 설치

`install.yaml`의 `custom_tools` 항목 순회:

1. **generate_image** (선택):
   - `gateway/tools/generate_image.py` 파일 존재 확인
   - 의존성 설치: `pip install python-dotenv google-genai --break-system-packages`
   - GEMINI_API_KEY 설정 안내:
     - 사용자에게 API Key 보유 여부 확인
     - 보유 시: `gateway/tools/.env` 파일에 `GEMINI_API_KEY=<key>` 저장
     - 미보유 시: Google AI Studio에서 발급 안내 (https://aistudio.google.com/apikey)
   - `required: false`이므로 실패해도 경고만 출력하고 계속 진행

### Step 5: Word 문서 의존성 설치

블로그 글을 Word(.docx)로 출력하기 위한 라이브러리 설치:
- `pip install python-docx --break-system-packages`

### Step 6: 적용 범위 선택

사용자에게 플러그인 적용 범위를 질문:

| 선택지 | 설명 | 대상 파일 |
|--------|------|----------|
| 모든 프로젝트 | 어디서든 이 플러그인 사용 | `~/.claude/CLAUDE.md` |
| 이 프로젝트만 | 현재 프로젝트에서만 사용 | `./CLAUDE.md` |

선택된 파일에 다음 내용 추가:
```
# blog-poster 플러그인
블로그 글 작성 요청 시 blog-poster 플러그인의 스킬을 활용하세요.
- `/blog-poster:write-post` — 블로그 글 작성 메인 워크플로우
- `/blog-poster:help` — 사용 가능한 명령 안내
```

### Step 7: 설치 결과 보고

설치 결과를 요약하여 사용자에게 보고:
- MCP 서버 설치 상태
- 커스텀 도구 설치 상태
- 의존성 설치 상태
- 적용 범위
- 다음 단계 안내: `/blog-poster:help`로 사용법 확인

[Top](#setup)

---

## 문제 해결

| 문제 | 해결 방법 |
|------|----------|
| context7 설치 실패 | `npx @upstash/context7-mcp@latest` 수동 실행 후 재시도 |
| GEMINI_API_KEY 미설정 | Google AI Studio에서 API Key 발급 후 `.env`에 설정 |
| python-docx 설치 실패 | `pip install python-docx --break-system-packages` 수동 실행 |
| 권한 오류 | `--user` 플래그 추가하여 사용자 레벨 설치 |

[Top](#setup)

---

## MUST 규칙

| # | 규칙 |
|---|------|
| 1 | `gateway/install.yaml`의 데이터만 참조 — 설치 대상을 하드코딩하지 않음 |
| 2 | `required: true` 항목 실패 시 설치를 중단하고 사용자에게 안내 |
| 3 | `required: false` 항목 실패 시 경고만 출력하고 계속 진행 |
| 4 | 이미 설치된 도구는 중복 설치하지 않음 (check 명령으로 확인) |
| 5 | API Key 등 민감 정보는 `.env` 파일에 저장하고 `.gitignore`에 포함 확인 |

[Top](#setup)

---

## MUST NOT 규칙

| # | 금지 사항 |
|---|----------|
| 1 | 사용자 확인 없이 자동으로 전역 설정을 변경하지 않음 |
| 2 | API Key를 코드나 로그에 노출하지 않음 |
| 3 | install.yaml에 없는 도구를 임의로 설치하지 않음 |

[Top](#setup)

---

## 검증 체크리스트

- [ ] `gateway/install.yaml`을 참조하여 설치를 수행하는가
- [ ] MCP 서버 설치 상태를 확인하고 중복 설치를 방지하는가
- [ ] `required: true` 항목 실패 시 중단 로직이 있는가
- [ ] API Key를 `.env`에 안전하게 저장하는가
- [ ] 적용 범위를 사용자에게 질문하는가
- [ ] 설치 결과를 요약 보고하는가

[Top](#setup)
