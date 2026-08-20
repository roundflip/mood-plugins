# mood-plugins

[mood](https://www.mood-by.ai) 의 AI 에이전트 플러그인 마켓플레이스입니다.

## 설치

mood-studio MCP 서버 주소는 `https://api.mood-by.ai/mcp` 입니다. 어떤 클라이언트든
연결하면 OAuth 로그인·동의 화면으로 이동하며, mood 계정으로 승인하면 저작 도구가
열립니다.

> 스킬(저작 프로세스·미디어·작법 가이드)은 **Claude Code 플러그인 설치 시에만**
> 함께 제공됩니다. 다른 클라이언트는 MCP 도구 11종만 연결됩니다.

### Claude Code (권장 — 스킬 포함)

```bash
/plugin marketplace add roundflip/mood-plugins
/plugin install mood-studio@mood-plugins
```

### Claude (claude.ai 웹 · 데스크톱)

1. **설정 → 커넥터(Connectors)** 로 이동
2. **커스텀 커넥터 추가(Add custom connector)** 선택
3. URL 에 `https://api.mood-by.ai/mcp` 입력 후 추가 — OAuth 설정은 비워둡니다
   (서버가 동적 등록을 지원합니다)
4. **연결(Connect)** 을 누르면 mood 로그인·동의 화면이 열립니다

Team/Enterprise 는 조직 관리자가 조직 설정 → 커넥터에서 먼저 추가해야 하며,
이후 멤버가 각자 연결합니다.

### Gemini CLI

```bash
gemini mcp add --transport http mood-studio https://api.mood-by.ai/mcp
```

연결 시 401 을 감지하면 OAuth 를 자동 탐색합니다. 인증 상태 확인·재로그인은
CLI 안에서 `/mcp auth mood-studio` 로 합니다.

### ChatGPT

1. **설정 → 보안 및 로그인(Security and login) → 개발자 모드(Developer mode)** 활성화
   (요금제·워크스페이스 정책에 따라 노출이 다를 수 있습니다)
2. 커넥터 추가 화면에서 **+** 를 눌러 이름·설명을 입력하고 서버 URL 에
   `https://api.mood-by.ai/mcp` 입력
3. 연결을 만들면 mood OAuth 로그인·동의로 이어지고, 서버가 노출하는 도구 목록을
   확인할 수 있습니다


## 플러그인 목록

| 플러그인 | 설명 |
|---|---|
| [`mood-studio`](./plugins/studio-plugin/) | 스튜디오 AI 저작 — MCP 도구 11종 + 저작·미디어·작법 스킬. 상세는 플러그인 README 참조 |

## 구조

```
mood-plugins/
├── .claude-plugin/marketplace.json   # Claude Code 마켓플레이스 카탈로그
└── plugins/
    └── studio-plugin/                # mood-studio 플러그인
        ├── .claude-plugin/ + .mcp.json    # Claude Code 네이티브 형식
        ├── plugin.json + mcp.json         # Agent Plugins 1.0.0 표준 형식
        └── skills/                        # 공용 (Agent Skills)
```

플러그인 릴리즈 시 `plugins/<이름>/` 의 `plugin.json`(양형식)과 이 카탈로그의
`version` 을 함께 올려야 사용자에게 자동 업데이트가 전달됩니다.
