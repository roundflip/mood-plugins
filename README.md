# mood-plugins

[mood](https://www.mood-by.ai) 의 AI 에이전트 플러그인 마켓플레이스입니다.

## 설치

마켓플레이스 주소는 이 리포의 GitHub 링크(`https://github.com/roundflip/mood-plugins`)
하나입니다. 설치 후 첫 도구 사용 시 mood 계정 OAuth 로그인·동의가 필요합니다.

### Claude Code

```bash
/plugin marketplace add roundflip/mood-plugins
/plugin install mood-studio@mood-plugins
```

### Claude (claude.ai 웹 · 데스크톱)

1. **설정 → 플러그인 → 마켓플레이스 추가** 에 GitHub 링크 입력
2. `mood-studio` 플러그인 설치

### ChatGPT

1. 사이드바의 **플러그인** 메뉴 → **마켓플레이스 추가** 에 GitHub 링크 입력
2. `mood-studio` 플러그인 설치


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
