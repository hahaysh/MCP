# VS Code에서 Confluence(Atlassian Rovo) MCP 연동 — **Python 기반** 실습 가이드

> 목표: **Python 프로젝트**를 VS Code에서 열고, **Confluence MCP 서버**를 등록한 뒤 **GitHub Copilot(에이전트 모드)**로 Confluence 작업(검색/조회/생성 등)을 수행합니다.

## 0. 사전 준비(Prerequisites)

* **Python 3.10+** (권장 3.11/3.12)
* **VS Code** + 확장:

  * *Python* (Microsoft)
  * *GitHub Copilot* (로그인 필수)
* **Git** (옵션: GitHub 계정/리포)
* **Atlassian Cloud**(Confluence 사용 가능) 계정
* (**권장**) Node.js 18+

  * Rovo MCP 인증/연결 편의를 위해 `mcp-remote`를 사용할 예정

---

## 1. Python 프로젝트 생성 & 가상환경 구성

```bash
# 작업 폴더 생성
mkdir confluence-mcp-python && cd confluence-mcp-python

# 가상환경 생성
python -m venv .venv

# (Windows PowerShell)
. .\.venv\Scripts\Activate.ps1
# (macOS/Linux)
source .venv/bin/activate

# 필수/선택 패키지 설치(예시)
pip install --upgrade pip
pip install requests python-dotenv
```

> 팁: `.venv/`는 `.gitignore`에 추가해 소스관리 대상에서 제외하세요.

---

## 2. VS Code로 열기 & 워크스페이스 준비

1. VS Code에서 현재 폴더 열기
2. Python 확장과 인터프리터(가상환경) 선택 확인
3. 기본 Python 스크립트 생성(테스트용)

```bash
ni app.py        # (Windows)
# 또는
touch app.py     # (macOS/Linux)
```

`app.py` 예시:

```python
def main():
    print("Hello, Confluence MCP on Python!")

if __name__ == "__main__":
    main()
```

---

## 3. Confluence MCP 서버 등록 (`.vscode/mcp.json`)

작업 루트에 **`.vscode`** 폴더를 만들고 **`mcp.json`** 파일을 생성합니다.

```bash
mkdir -p .vscode
```

`.vscode/mcp.json` (권장: `mcp-remote` 프록시 방식)

```json
{
  "servers": {
    "atlassian-rovo": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.atlassian.com/v1/sse"]
    }
  }
}
```

> 왜 `mcp-remote`인가요?
> 브라우저 OAuth 인증·SSE 연결을 프록시가 처리해 VS Code에서 **원격 Rovo MCP 서버**와 손쉽게 연동됩니다.

---

## 4. MCP 서버 시작 & 로그인

1. `mcp.json` 저장 후 VS Code 우측 상단(또는 파일 상단)에서 **Start** 버튼 클릭
2. 브라우저가 열리면 **Atlassian 로그인/승인** 진행
3. VS Code에 서버 상태가 **Running**으로 보이면 성공

> 토큰 만료/에러 시 **Restart**로 재로그인하면 됩니다.

---

## 5. Copilot(에이전트 모드)에서 도구 사용

1. VS Code **Chat** 열기 → 상단에서 **Agent** 모드 선택
2. **Tools(🔧)** 버튼 → `MCP Server: atlassian-rovo` 도구 활성화 확인
3. 아래처럼 **자연어 지시**를 입력하면 필요 시 MCP 도구가 자동 호출됩니다.

### 예시 프롬프트(파이썬 업무 맥락)

* **문서 검색(온보딩/런북)**

  ```
  Search Confluence pages about "Python service onboarding" in space ENG
  and list the top 5 with links.
  ```
* **회의록 템플릿 기반 페이지 생성**

  ```
  Create a Confluence page in space DEV titled "API Meeting Notes - {today}"
  including sections: Agenda, Decisions, Action Items (with a task list).
  ```
* **배포/런북 연결**

  ```
  Find pages labeled "runbook" mentioning "uvicorn" or "gunicorn".
  Summarize the restart procedure and link them here.
  ```
* **PR/릴리스 노트와 연계 서술**

  ```
  Draft a Confluence page "Release Notes for v0.3.0" summarizing merged PRs
  from the last 2 weeks in repo myorg/py-service. Include a changelog table.
  ```

---

## 6. Python 개발 루틴과 함께 쓰는 흐름(추천 워크플로우)

1. **코딩**: 가상환경에서 기능 구현
2. **지식화**: Confluence에 설계/결정/런북을 **템플릿+라벨**로 표준화 등록
3. **자동화**: Copilot(에이전트)이 MCP로 페이지 생성·요약·목차/링크 정리를 지원
4. **릴리스**: 태깅/PR 머지 후 Confluence에 **릴리스 노트/변경 이력** 페이지 자동 작성 보조
5. **운영**: 장애/운영 이슈 발생 시 관련 문서/런북을 **빠르게 검색**하고 팀 공유

---

## 7. 트러블슈팅

* **서버가 시작되지 않음**

  * Node.js 18+ / `npx` 사용 가능 여부 확인
  * `mcp.json` JSON 문법 오류 검사
  * VS Code에서 **MCP: List/Show Servers**로 상태 확인
* **도구가 안 보임/호출 안 됨**

  * Chat이 **Agent 모드**인지 확인
  * Tools(🔧)에서 `atlassian-rovo`가 **체크**되어 있는지 확인
* **인증 팝업이 안 뜸**

  * **Restart** 시도 → 방화벽/조직 SSO 정책 확인
* **보안/데이터 정책**

  * 공식 Rovo MCP(Atlassian)만 사용 권장
  * 사내 규정상 외부 연결 제한 시 보안팀 확인

---

## 8. (선택) 팀 공유/재현을 위한 설정

* **레포에 `.vscode/mcp.json` 포함**:
  팀원이 같은 리포를 열면 동일한 MCP 구성을 바로 사용
* **Dev Containers**:
  `devcontainer.json` → `customizations.vscode.mcp.servers`에 동일 항목을 넣어 컨테이너 생성 시 자동 반영
* **전역(User) 설정**:
  여러 프로젝트에서 쓰려면 **MCP: Open User Configuration**로 전역 `mcp.json`에 등록

---

## 9. 부록: Python 샘플(선택)

MCP 없이 **Confluence REST**를 직접 호출하고 싶다면(학습용):

```python
# confluence_client.py
import os
import requests
from requests.auth import HTTPBasicAuth
from dotenv import load_dotenv

load_dotenv()

BASE_URL = os.getenv("CONFLUENCE_URL")  # e.g., https://your-domain.atlassian.net/wiki
EMAIL = os.getenv("ATLASSIAN_EMAIL")
API_TOKEN = os.getenv("ATLASSIAN_API_TOKEN")

def search_pages(cql: str, limit: int = 10):
    url = f"{BASE_URL}/rest/api/search"
    params = {"cql": cql, "limit": limit}
    resp = requests.get(url, params=params, auth=HTTPBasicAuth(EMAIL, API_TOKEN))
    resp.raise_for_status()
    return resp.json()

if __name__ == "__main__":
    # 예: 라벨이 'runbook'인 페이지 5개
    data = search_pages('label = "runbook"', limit=5)
    for r in data.get("results", []):
        print(r.get("title"))
```

> 실전에서는 **MCP를 통해 Copilot이 자동으로 문서 작업을 도와주게** 하는 것이 이 가이드의 핵심입니다. 위 코드는 Python REST 호출 감을 익히는 *참고용*입니다.

---

## 10. 마무리(요점 정리)

* Python 가상환경에서 개발 → VS Code에서 **Confluence MCP** 등록
* **Start → 로그인 승인 → Running**
* Copilot **에이전트 모드 + Tools**로 Confluence 작업을 자연어로 수행
* 팀 문서(설계/회의록/런북/릴리즈)를 **표준화 + 자동화**하여 생산성 향상

---
