# mood-studio

[mood](https://www.mood-by.ai) 스튜디오의 **AI 저작 플러그인**입니다. Claude Code
등 MCP 를 지원하는 에이전트 환경에서 mood 세계관(작품)을 만들고 편집·게시할 수
있습니다.

- **MCP 서버**: `https://api.mood-by.ai/mcp` — 세계관 생성·드래프트 편집·검증·
  게시·태그·미디어 업로드 도구 11종
- **스킬 3종**: 저작 워크플로우 / 미디어 업로드 / 콘텐츠 작법

두 가지 플러그인 형식을 한 리포에 담고 있습니다:

| 형식 | 파일 | 대상 클라이언트 |
|---|---|---|
| [Agent Plugins 1.0.0](https://agent-plugins.org) 표준 | 루트 `plugin.json` + `mcp.json` | Codex, Cursor, GitHub Copilot/VS Code 등 표준 채택 클라이언트 |
| Claude Code 네이티브 | `.claude-plugin/plugin.json` + `.mcp.json` | Claude Code |

`skills/`(Agent Skills 형식)는 양쪽이 공유합니다.

## 설치 — Claude Code (플러그인)

[mood-plugins 마켓플레이스](../../README.md)를 통해 설치합니다:

```bash
/plugin marketplace add roundflip/mood-plugins
/plugin install mood-studio@mood-plugins
```

플러그인이 MCP 서버 연결과 스킬을 함께 설치합니다. 첫 도구 사용 시 브라우저가
열리며 mood 로그인·권한 동의를 요청합니다 — mood 계정(가입 회원)이 필요합니다.

> 업데이트 배포 시에는 양형식 `plugin.json` 과 마켓플레이스 카탈로그의
> `version` 을 함께 올려야 사용자에게 자동 업데이트가 전달됩니다.

## 설치 — 수동 MCP 등록 (Claude Code · Cursor 등)

플러그인 없이 MCP 서버만 등록해도 도구는 전부 사용할 수 있습니다 (스킬의
워크플로우 가이드는 빠집니다).

```bash
# Claude Code
claude mcp add --transport http mood-studio https://api.mood-by.ai/mcp
```

```json
// Cursor 등 — mcp 설정 파일에 추가
{
  "mcpServers": {
    "mood-studio": { "type": "http", "url": "https://api.mood-by.ai/mcp" }
  }
}
```

인증은 MCP 표준 OAuth 를 따릅니다 — 클라이언트가 자동으로 브라우저를 열어
로그인·동의를 진행합니다. 별도 API 키나 client id 설정은 필요 없습니다.

## 권한 (동의 화면에서 승인하는 범위)

| 스코프 | 의미 |
|---|---|
| `studio` | 나의 세계관 읽기·쓰기 (드래프트 편집·검증) |
| `studio:publish` | 나의 세계관 게시 |
| `media` | 미디어 업로드 (이미지, 비디오) |

연결 해제는 언제든 클라이언트에서 서버를 제거하면 됩니다.

## 스킬

| 스킬 | 내용 |
|---|---|
| `studio-authoring` | 생성 → 드래프트 편집(전체 스냅샷·revision 규약) → 검증 → 게시 워크플로우 |
| `studio-media` | presigned 2단계 이미지 업로드와 usage 자리 규칙 |
| `studio-writing` | 세계관·캐릭터·시나리오 작법 — 각 필드가 플레이 AI 의 어떤 재료가 되는지 |

## 주의

- **게시는 공개 행위**입니다 — 에이전트는 게시 전 반드시 사용자에게 확인해야
  합니다 (`studio-authoring` 스킬에 규칙으로 포함).
- 도구 호출로 만들어지는 작품·미디어는 전부 로그인한 본인 계정 소유입니다.
