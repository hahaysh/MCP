## 🎯 실습 가이드: VS Code + Confluence 자체 Python MCP 서버 연동 단계별

---

### 준비: 필수 사항 점검

✔ VS Code 설치 + GitHub Copilot 확장 설치 및 로그인
✔ Python ≥ 3.10 설치 (3.11/3.12 권장)
✔ Atlassian Cloud 계정 (Confluence 사용 가능)
✔ Confluence API 토큰 발급 (계정 이메일과 함께 사용)
✔ 조직 네트워크나 방화벽 정책이 Confluence API 호출을 허용하는지 확인

---

### 1단계: Confluence 연결 정보 확인하기

이 단계에서는 `mcp.json` 설정에 입력할 **Confluence URL, 사용자 이메일(Username), API 토큰**을 미리 준비합니다.

#### 1. Confluence URL

* 사용 중인 [Confluence 사이트](https://www.atlassian.com/ko/software/confluence)에 로그인했을 때 브라우저 주소창에 표시되는 **기본 URL**입니다.
* 메모장 등에 복사해 두세요.
* **예시**: `https://your-domain.atlassian.net/wiki`

#### 2. Confluence 사용자 이름 (Username)

* Confluence(Atlassian) 계정에 로그인할 때 사용하는 **본인 이메일 주소**입니다.
* 메모장 등에 복사해 두세요.
* **예시**: `your-email@example.com`

#### 3. Confluence API 토큰 (API Token)

API 토큰은 비밀번호 대신 프로그램이 Confluence에 안전하게 접근할 수 있도록 해주는 인증 키입니다.

1. 브라우저에서 Atlassian API 토큰 발급 페이지 접속 → [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens)
2. **Create API token** 버튼 클릭
3. 이름(Label)을 입력 (예: `python-mcp-token`)
4. **Create** 버튼 클릭 → 새 토큰 생성
5. 화면에 나타난 토큰을 **즉시 복사 및 안전한 곳에 저장**

   > ⚠️ 토큰은 화면을 닫으면 다시 볼 수 없으니 반드시 안전하게 보관하세요.

---

> ✅ 준비된 URL, 이메일, API 토큰은 VS Code와 Confluence 사이에서 **자체 Python MCP 서버**의 REST API 인증에 사용됩니다.

---

### 2단계: Python MCP 서버 코드 준비

> ✅ 이후의 진행은 codespace에서 진행을 합니다.
> 작업은 dev20250926 브랜치를 만들어서 작업합니다. `git switch -c dev20250926`

프로젝트 폴더 안에 `server_confluence.py` 파일을 생성하고 아래 코드를 붙여 넣으세요:

```python
import os
import requests
from fastapi import FastAPI
from pydantic import BaseModel
from requests.auth import HTTPBasicAuth
from dotenv import load_dotenv

# .env 파일 로드
load_dotenv()

app = FastAPI(title="Confluence MCP Server")

CONFLUENCE_URL = os.getenv("CONFLUENCE_URL")
EMAIL = os.getenv("ATLASSIAN_EMAIL")
API_TOKEN = os.getenv("ATLASSIAN_API_TOKEN")

auth = HTTPBasicAuth(EMAIL, API_TOKEN)

class SearchRequest(BaseModel):
    cql: str
    limit: int = 5

class GetPageRequest(BaseModel):
    page_id: str

class CreatePageRequest(BaseModel):
    space: str
    title: str
    body: str
    parent_id: str | None = None

@app.post("/tools/search_pages")
def search_pages(req: SearchRequest):
    url = f"{CONFLUENCE_URL}/rest/api/search"
    resp = requests.get(url, params={"cql": req.cql, "limit": req.limit}, auth=auth)
    resp.raise_for_status()
    return resp.json()

@app.post("/tools/get_page")
def get_page(req: GetPageRequest):
    url = f"{CONFLUENCE_URL}/rest/api/content/{req.page_id}"
    params = {"expand": "body.storage,version"}
    resp = requests.get(url, params=params, auth=auth)
    resp.raise_for_status()
    return resp.json()

@app.post("/tools/create_page")
def create_page(req: CreatePageRequest):
    url = f"{CONFLUENCE_URL}/rest/api/content/"
    data = {
        "type": "page",
        "title": req.title,
        "space": {"key": req.space},
        "body": {"storage": {"value": req.body, "representation": "storage"}}
    }
    if req.parent_id:
        data["ancestors"] = [{"id": req.parent_id}]
    resp = requests.post(url, json=data, auth=auth)
    resp.raise_for_status()
    return resp.json()
```

---

### 3단계: Python 환경 설정 & 서버 실행

1. 가상환경 생성 및 패키지 설치

   ```bash
   python -m venv .venv
   source .venv/bin/activate   # (Windows: .venv\Scripts\activate)
   pip install fastapi uvicorn requests pydantic python-dotenv
   ```

   > ✅ `python-dotenv` 패키지를 추가 설치해야 `.env` 파일을 불러올 수 있습니다.

2. 환경 변수 등록 대신 `.env` 파일을 준비합니다.
   (`.env` 파일은 프로젝트 루트에 위치)

   ```
   CONFLUENCE_URL=https://your-domain.atlassian.net/wiki
   ATLASSIAN_EMAIL=you@example.com
   ATLASSIAN_API_TOKEN=your_api_token
   ```

   > 더 이상 `export`나 `set` 명령어로 직접 등록할 필요가 없습니다.

3. 서버 실행

   ```bash
   uvicorn server_confluence:app --host 0.0.0.0 --port 8000
   ```

   실행 후 `http://localhost:8000/docs`에 접속하면 API 문서를 확인할 수 있습니다.

---

### 4단계: VS Code `mcp.json` 설정

프로젝트 루트 `.vscode/mcp.json` 파일에 아래를 작성합니다:

```json
{
  "servers": {
    "confluence-local": {
      "type": "http",
      "url": "http://localhost:8000"
    }
  }
}
```

> ✅ 이제 VS Code에서 MCP 서버를 **로컬 Python 서버**로 인식하게 됩니다.

---

### 5단계: Copilot 에이전트 모드 설정 & 도구 확인

1. VS Code에서 **Copilot Chat** 열기
2. 상단 드롭다운에서 **Agent 모드** 선택
3. **Tools(🔧)** → `MCP Server: confluence-local` 확인 및 활성화

---

### 6단계: 예시 프롬프트로 기능 테스트

* 페이지 검색:

  ```
  Search Confluence pages about "python onboarding" in space HR
  ```

* 페이지 조회:

  ```
  Get the Confluence page with ID 1234567
  ```

* 페이지 생성:

  ```
  Create a Confluence page in space ENG titled "Lab Notes"
  with sections: Summary and Action Items
  ```

---

### 7단계: 문제 & 해결 체크리스트

| 문제               | 원인 가능성                         | 해결 방법                                                |
| ---------------- | ------------------------------ | ---------------------------------------------------- |
| 서버 시작 안 됨        | 패키지 설치 누락, 포트 충돌               | `pip install fastapi uvicorn requests` 확인 / 다른 포트 사용 |
| MCP 도구 안 보임      | Agent 모드 미선택, `mcp.json` 문법 오류 | Agent 모드 확인 / `mcp.json` 다시 저장                       |
| 401 Unauthorized | 이메일·토큰 값 불일치                   | 환경 변수 값 다시 확인 / 토큰 재발급                               |
| 403 Forbidden    | API 권한 부족                      | 계정 권한 확인 (Confluence 관리자 문의)                         |

---

### 8단계: 확장 아이디어

* `update_page`, `list_spaces` 등 추가 도구 구현
* Dockerfile로 MCP 서버 컨테이너화 → 팀 내 공유
* DevContainer `customizations.vscode.mcp.servers`에 등록하여 자동 설정

---

✅ 이렇게 하면 **Node.js, mcp-remote, OAuth 로그인 없이** 순수 Python + API 토큰 기반으로 Confluence를 Copilot과 연결할 수 있습니다.

---

원하시면 이 버전을 **PPT 발표용 5~6장 요약**으로도 정리해드릴까요?
