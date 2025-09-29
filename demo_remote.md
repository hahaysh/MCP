# VS Code + Confluence(Atlassian Rovo) MCP 서버 연동 가이드

> 목표: VS Code에서 Atlassian Rovo **MCP 서버**를 등록하고, **GitHub Copilot(Agent 모드)**에서 Confluence 관련 작업(검색/조회/생성 등)을 실행합니다.
> MCP 설정 파일은 **`.vscode/mcp.json`** 를 사용합니다. ([Visual Studio Code][1])

## 준비물(Prerequisites)

* **VS Code 1.102+** 및 **GitHub Copilot** 확장 설치(로그인 포함) ([Visual Studio Code][1])
* **Atlassian Cloud** 사이트(Confluence 사용 가능) 계정
* **Node.js v18+** (VS Code가 `npx mcp-remote`를 실행할 때 필요) ([Atlassian Support][2])
* (조직 정책) 외부 MCP 사용 허용 여부 확인

---

## 1) 워크스페이스 열기 & 설정 파일 위치 만들기

1. VS Code로 실습용 폴더(demo00 폴더 또는 Git 저장소)를 엽니다.
2. 루트에 **`.vscode`** 폴더가 없다면 생성하고, 그 안에 **`mcp.json`** 파일을 만듭니다.

   * 이 방식은 **레포지토리 단위 공유**에 유리합니다(같은 폴더를 연 사람은 동일 설정 사용). ([GitHub Docs][3])

---

## 2) Atlassian Rovo MCP 서버 등록(권장: mcp-remote 프록시)

Atlassian이 안내하는 권장 방법은 **`mcp-remote`** 프록시를 통해 원격 Rovo MCP 서버(`https://mcp.atlassian.com/v1/sse`)에 연결하는 것입니다. VS Code는 아래처럼 **표준 입출력(stdio)** 방식으로 프로세스를 띄울 수 있습니다. ([Atlassian Support][2])

`.vscode/mcp.json`:

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

> 왜 `mcp-remote`?
> OAuth 인증과 SSE 통신을 **프록시가 대신 처리**해 주어 VS Code/에디터 연동이 쉬워집니다(베타). ([Atlassian Support][2])

---

## 3) 서버 시작 & 로그인

1. `mcp.json`을 저장하면 파일 상단에 **Start/Restart** 액션이 나타납니다. **Start**를 클릭하세요. ([Visual Studio Code][1])
2. 브라우저가 열리며 Atlassian 로그인 및 권한 승인을 진행합니다(OAuth).
3. 성공하면 VS Code에 **Running** 상태로 표시됩니다.

> 팁: 인증 토큰이 만료되면 **서버 재시작**으로 다시 로그인하면 됩니다. ([Atlassian Support][2])

---

## 4) Copilot에서 도구 확인(Agent 모드)

1. VS Code **Chat 뷰**를 열고, 상단 드롭다운에서 **Agent** 모드로 전환합니다.
2. **Tools(🔧) 버튼**을 눌러 사용 가능한 도구 목록을 확인하고, `MCP Server: atlassian-rovo` 관련 도구를 선택합니다.
3. 이제 채팅 입력창에서 자연어로 지시하면, 필요 시 MCP 도구가 자동 호출됩니다. ([Visual Studio Code][1])

---

## 5) 바로 써보는 예시 프롬프트

> 실제 제공 도구 이름은 서버/버전별로 조금 다를 수 있습니다. 아래는 “형태 예시”입니다.

* **페이지 검색**
  `Search Confluence pages about "onboarding checklist" in space HR.`
* **페이지 가져오기**
  `Get the Confluence page by title "2025-Q4 OKRs" in space OPS.`
* **새 페이지 생성(요약 포함)**
  `Create a Confluence page in space ENG titled "MCP Trial Notes" with a summary section and a task list.`
* **링크/첨부 확인**
  `List attachments and outgoing links for the current page.`

---

## 6) (선택) 원격 서버 URL 직접 등록 방식

일부 클라이언트는 아래처럼 **원격 서버 URL을 직접** 등록하기도 합니다. VS Code는 `http/sse` 타입을 지원합니다. 다만 **OAuth 흐름 때문에 `mcp-remote` 방식이 더 호환성 좋습니다.** ([Visual Studio Code][1])

```json
{
  "servers": {
    "atlassian-rovo": {
      "type": "sse",
      "url": "https://mcp.atlassian.com/v1/sse"
    }
  }
}
```

---

## 7) 트러블슈팅

* **서버가 안 뜨거나 금방 꺼짐**

  * Node.js 18+ / `npx` 사용 가능 여부 확인
  * `mcp.json` JSON 문법 오류 확인
  * **MCP: Show Installed Servers** / **List Servers**로 상태 확인 ([Visual Studio Code][1])
* **도구가 보이지 않음**

  * Chat에서 **Agent 모드**인지, Tools에서 해당 서버 도구가 **선택**되어 있는지 확인 ([Visual Studio Code][1])
* **인증 팝업이 안 뜸/권한 오류**

  * VS Code에서 서버 **Restart**
  * 조직의 Atlassian/SSO 정책 확인(Cloud 베타 기능) ([Atlassian Support][4])
* **보안 우려(서드파티 서버 사용)**

  * Rovo(Atlassian) 공식 원격 서버만 사용하거나, 커뮤니티 서버 사용 시 **코드/권한**을 반드시 검토하세요. ([Atlassian Support][4])

---

## 8) 정리(요점)

* `.vscode/mcp.json`에 Atlassian Rovo MCP를 등록
* **Start → 로그인 승인 → Running**
* Copilot **Agent 모드 + Tools**에서 Confluence 작업 실행
* 필요 시 PR/이슈 자동화 등 **팀 협업 플로우**로 확장

---

## 부록 A) 전역 설정으로 쓰기

여러 프로젝트에서 공통으로 쓰고 싶다면, **MCP: Open User Configuration** 명령으로 **사용자 전역 `mcp.json`**에 추가할 수 있습니다. (워크스페이스/전역/리모트별 저장 위치 지원) ([Visual Studio Code][1])

## 부록 B) Dev Containers에 포함시키기

`devcontainer.json`의 `customizations.vscode.mcp.servers`에 동일한 구성을 넣으면, 컨테이너 생성 시 자동 반영됩니다. ([Visual Studio Code][1])

---

### 참고 문서

* **VS Code: Use MCP servers** — 설정 위치, 형식, Agent 모드/Tools 사용법, Dev Container 연동 등 공식 가이드. ([Visual Studio Code][1])
* **Atlassian Rovo MCP Server: IDE 설정** — `mcp-remote` 사용, SSE 엔드포인트, 설치 팁(베타). ([Atlassian Support][2])
* **Atlassian Rovo MCP Server: 소개/베타 안내** — 지원 클라이언트, 보안 모델, 사용 범위. ([Atlassian Support][4])

---

[1]: https://code.visualstudio.com/docs/copilot/customization/mcp-servers "Use MCP servers in VS Code"
[2]: https://support.atlassian.com/atlassian-rovo-mcp-server/docs/setting-up-ides/?utm_source=chatgpt.com "Setting up IDEs (desktop clients) | Atlassian Rovo MCP ..."
[3]: https://docs.github.com/copilot/customizing-copilot/using-model-context-protocol/extending-copilot-chat-with-mcp?utm_source=chatgpt.com "Extending GitHub Copilot Chat with the Model Context ..."
[4]: https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/?utm_source=chatgpt.com "Getting started with the Atlassian Rovo MCP Server"
