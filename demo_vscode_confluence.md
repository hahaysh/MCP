## 🎯 실습 가이드: VS Code + Confluence Rovo MCP 서버 연동 단계별

---

### 준비: 필수 사항 점검

✔ VS Code 설치 + GitHub Copilot 확장 설치 및 로그인
✔ Node.js ≥ v18 설치
✔ Atlassian Cloud 계정 (Confluence 사용 가능)
✔ 조직 네트워크나 방화벽 정책이 외부 MCP 접속을 허용하는지 확인

---

### 1단계: API 토큰 발급하기 (Atlassian)

| 순서 | 실행 항목                                                                   | 비고                                              |
| -- | ----------------------------------------------------------------------- | ----------------------------------------------- |
| 1  | 브라우저로 이동: `https://id.atlassian.com/manage-profile/security/api-tokens` | Atlassian 계정 보안 설정 페이지로 이동                      |
| 2  | **Create API token** 또는 **Create API token with scopes** 클릭             | 토큰 생성 화면 열림                                     |
| 3  | 토큰 이름 입력, 만료 기간(1~365일) 선택                                              | 식별하기 쉬운 이름 권장                                   |
| 4  | **Create / Generate** 클릭 → 새 토큰 생성                                      | 이때 토큰이 화면에 표시됨                                  |
| 5  | 화면에 나타난 토큰을 **즉시 복사 및 안전한 곳에 저장**                                       | 토큰은 다시 볼 수 없으므로 보관해야 함 ([Atlassian Support][1]) |

> ✅ 이 API 토큰은 나중에 IDE/서버 설정에서 이메일과 조합하여 인증용으로 사용됩니다.

---

### 2단계: VS Code 워크스페이스 열기 & `mcp.json` 파일 준비

1. VS Code에서 실습용 프로젝트 폴더 열기
2. 루트 디렉터리에 `.vscode` 폴더가 없다면 생성
3. `.vscode/mcp.json` 파일 생성

---

### 3단계: `.vscode/mcp.json` 내용 채우기 (mcp-remote 방식 권장)

아래 내용을 복사해서 `mcp.json`에 붙여 넣고 저장하세요:

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

* `mcp-remote`는 OAuth 인증 및 SSE 연결을 프록시가 대신 처리해 줍니다. ([Atlassian Support][2])
* 저장하면 VS Code 내에서 **Start / Restart** 버튼이 나타납니다.

---

### 4단계: MCP 서버 시작 및 인증 흐름

1. `mcp.json` 저장 후, VS Code에서 **Start** 버튼 클릭
2. 브라우저가 팝업되며 Atlassian 로그인/권한 승인 화면이 뜸
3. 로그인 & 승인을 완료하면 VS Code에서 MCP 서버가 **Running** 상태로 표시됨
4. 만약 인증 실패나 토큰 만료 등이 발생하면 **Restart** 버튼으로 다시 실행

---

### 5단계: Copilot 에이전트 모드 설정 & 도구 확인

1. VS Code에서 **Copilot Chat** 뷰 열기
2. 드롭다운 메뉴에서 **Agent 모드** 선택
3. **Tools (🔧)** 버튼 클릭 → `MCP Server: atlassian-rovo` 도구가 보이는지 확인
4. 도구가 보이지 않으면 체크표시해서 활성화

---

### 6단계: 예시 프롬프트로 기능 테스트

아래 예시를 그대로 복사해 Chat 입력창에 넣고 실행해 보세요:

* 페이지 검색:

  > `Search Confluence pages about "onboarding checklist" in space HR and list top 5 with links.`
* 페이지 단건 조회:

  > `Get the Confluence page by title "2025-Q4 OKRs" in space OPS.`
* 페이지 생성:

  > `Create a Confluence page in space ENG titled "MCP Trial Notes" with sections: Summary, Decisions, Action Items.`
* 문서 링크/첨부 보기:

  > `List attachments and outgoing links for the current page.`

---

### 7단계: 발생 가능한 문제 & 해결 체크리스트

| 문제                | 원인 가능성                         | 해결 방법                          |
| ----------------- | ------------------------------ | ------------------------------ |
| 서버 시작 안 됨 / 바로 종료 | Node 버전 낮거나 `npx` 실행 불가        | Node v18+ 설치 / `npx` 정상 동작 확인  |
| MCP 도구 안 보임       | Agent 모드 선택 안 함 또는 Tools 비활성화  | Agent 모드 선택 / Tools 메뉴에서 서버 체크 |
| 인증 팝업 안 뜸         | 브라우저 팝업 차단 또는 조직 SSO 제한        | 팝업 허용 / 조직 정책 확인 / Restart 시도  |
| 권한/접근 오류          | Atlassian 계정 권한 부족 또는 토큰 권한 축소 | 계정 권한 확인 / 토큰 재발급              |

---

### 8단계: (선택) 사용자 전역 설정 또는 컨테이너 설정

* 여러 프로젝트에서 같은 MCP 서버를 쓰려면 VS Code에서 **MCP: Open User Configuration** 후 전역 `mcp.json` 설정
* Dev Container 환경에서는 `devcontainer.json` 내 `customizations.vscode.mcp.servers` 항목에 동일 MCP 설정 추가

---

이렇게 단계별로 순차적으로 따라가면, 학생들이 손쉽게 **Atlassian API 토큰 발급 → VS Code MCP 서버 등록 → Copilot에서 Confluence 기능 사용**까지 실행해 볼 수 있을 거예요.

필요하시면 이걸 PPT 버전으로 바꿔 드릴까요?

[1]: https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/?utm_source=chatgpt.com "Manage API tokens for your Atlassian account"
[2]: https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/?utm_source=chatgpt.com "Getting started with the Atlassian Rovo MCP Server"
