# mood-plugins

[mood](https://www.mood-by.ai) 의 AI 에이전트 플러그인 마켓플레이스입니다.

## 설치 — Claude Code

```bash
/plugin marketplace add roundflip/mood-plugins
/plugin install mood-studio@mood-plugins
```


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
