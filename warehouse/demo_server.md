# 🔧 부록: 도메인 내부에서 Confluence MCP 서버 직접 구현하기

## 1. 왜 자체 서버를 구현하나요?

* **사내 도메인 보안 정책** 때문에 외부 Atlassian Rovo MCP 서버(`https://mcp.atlassian.com/v1/sse`)를 직접 쓰기 어려울 수 있습니다.
* 이 경우, **Confluence REST API**를 직접 호출하는 **MCP 서버**를 사내 인프라에 띄우고, Copilot은 그 MCP 서버를 호출하도록 구성하면 됩니다.

---

## 2. MCP 서버 기본 구조 (Python)

아래는 **Python 기반 FastAPI**로 Confluence REST API를 MCP 표준 규격에 맞춰 감싸는 최소 예시입니다.

```python
# server_confluence.py
import os
import requests
from fastapi import FastAPI
from pydantic import BaseModel
from requests.auth import HTTPBasicAuth

app = FastAPI()

CONFLUENCE_URL = os.getenv("CONFLUENCE_URL", "https://your-company.atlassian.net/wiki")
USERNAME = os.getenv("ATLASSIAN_EMAIL", "you@example.com")
API_TOKEN = os.getenv("ATLASSIAN_API_TOKEN", "your_api_token")

auth = HTTPBasicAuth(USERNAME, API_TOKEN)

# === MCP 도구: 페이지 검색 ===
class SearchRequest(BaseModel):
    cql: str
    limit: int = 5

@app.post("/tools/search_pages")
def search_pages(req: SearchRequest):
    url = f"{CONFLUENCE_URL}/rest/api/search"
    resp = requests.get(url, params={"cql": req.cql, "limit": req.limit}, auth=auth)
    resp.raise_for_status()
    return resp.json()
```

> 이 서버는 **MCP 표준**에 맞게 `/tools/...` 엔드포인트를 노출하여 Copilot이 사용할 수 있는 “도구(tools)”로 등록됩니다.

---

## 3. 로컬 실행

```bash
uvicorn server_confluence:app --host 0.0.0.0 --port 8000
```

> Docker로 감싸 배포하면 내부 네트워크에서 컨테이너 형태로도 운영 가능합니다.

---

## 4. VS Code MCP 설정 (`.vscode/mcp.json`)

사내 MCP 서버를 등록하는 예시입니다.

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

* `url` 부분을 사내 도메인 서버 주소로 교체하면, 팀 전체가 같은 MCP 서버를 공유할 수 있습니다.
* 예: `"url": "http://mcp.confluence.mycorp.local:8000"`

---

## 5. GitHub Copilot에서 활용

1. VS Code에서 `mcp.json`을 저장 → **Start** 버튼으로 서버 시작
2. Copilot Chat (에이전트 모드) → Tools(🔧) → `MCP Server: confluence-local` 선택
3. 이제 Confluence 관련 프롬프트 실행 시 MCP 서버가 내부 REST API를 호출하게 됩니다.

예시:

```
Search for pages with label "runbook" mentioning "python"
```

---

## 6. 운영 시 고려사항

* **인증**: API 토큰을 환경 변수/시크릿 매니저(K8s Secret, Vault 등)에 저장
* **보안**: HTTPS 적용 + 내부 인증(사내 SSO, JWT 등)
* **확장**: `/tools/create_page`, `/tools/get_page`, `/tools/list_spaces` 등 REST API 매핑
* **배포**: Docker/K8s로 컨테이너화 → 도메인 내 MCP Gateway로 서비스

---

👉 요약하자면:

* 외부 MCP 서버를 쓰기 힘든 환경에서는 **Confluence REST API를 감싼 Python FastAPI MCP 서버**를 직접 띄운다.
* VS Code `mcp.json`에서 그 서버를 등록한다.
* Copilot이 내부 MCP 서버를 통해 안전하게 Confluence를 사용할 수 있다.

---
