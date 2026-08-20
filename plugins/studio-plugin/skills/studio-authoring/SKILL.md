---
name: studio-authoring
description: mood 스튜디오에서 세계관(월드)을 만들고 편집·게시할 때 사용. mood-studio MCP 도구(create_world, get/save_world_draft, validate_world_draft, publish_world, 태그)의 올바른 호출 순서·드래프트 스냅샷 규약·게시 게이트를 다룬다. 사용자가 "mood에 작품/세계관을 만들어줘", "드래프트 수정", "게시해줘"라고 하면 이 스킬을 따른다.
---

# mood 스튜디오 저작 워크플로우

mood-studio MCP 서버의 도구로 세계관(월드)을 저작한다. 처음 도구를 호출하면
브라우저에서 mood 로그인·권한 동의가 열린다 — 사용자가 동의를 마쳐야 도구를
쓸 수 있다.

## 핵심 모델 — 드래프트가 정본이다

- 작품의 모든 것(제목·소개·커버·태그·공개범위·세계관·장소·캐릭터·사건·시나리오)은
  **드래프트 스냅샷 하나**에 담긴다. `get_world_draft` 로 통째로 받고,
  편집한 뒤 `save_world_draft` 로 **통째로** 돌려보낸다 — 부분 저장이 아니다.
- 게시(`publish_world`) 전까지 드래프트는 플레이어에게 보이지 않는다. 게시 후
  다시 편집하면 새 드래프트가 만들어져 공개본과 분리된다.
- **저장은 관대하고, 게시는 엄격하다.** 빈 제목·화자 없는 대사·장소 없는 사건도
  저장은 된다. 완성도는 게시 시점에만 검증된다. 단 **존재하지 않는 대상을
  가리키는 참조**(없는 장소 id, 남의 미디어 id 등)는 저장에서도 거부된다.

## 표준 흐름

```
create_world                       → world_id 획득 (빈 초안 + PLAYER 캐릭터 자동 생성)
get_world_draft(world_id)          → { draft, revision }
(draft 편집)
save_world_draft(world_id, draft, revision)   → { draft(id 발급 반영), revision+1 }
...반복...
validate_world_draft(world_id)     → violations 확인 (빈 배열 = 게시 가능)
publish_world(world_id)            → 게시 (반드시 사용자에게 먼저 확인!)
```

## 저장 규약 — 반드시 지킬 것

1. **id 는 문자열이다.** 모든 id(`world_id`, 엔티티 id, `cover_media_id`,
   `tag_ids`)는 JSON 문자열로 다룬다 — 숫자로 바꾸면 정밀도가 깨진다.
2. **신규 항목은 `id` 필드를 아예 생략**하고 `clientRef`(임의 문자열, 64자
   이내)를 채운다. `"id": "0"` 을 보내면 즉시 거부된다. 서버가 id 를 발급해
   응답 draft 에 같은 `clientRef` 와 함께 돌려준다 — 다음 저장부터는 그 id 를
   쓴다.
3. **같은 저장 안에서 신규 항목을 참조**하려면 `placeClientRef`(사건→장소),
   `characterClientRef`(시나리오 대사→화자)를 쓴다. 신규 캐릭터와 그 캐릭터가
   말하는 시나리오를 한 번에 저장할 수 있다.
4. **`revision` 은 낙관적 락**이다. `get_world_draft`/`save_world_draft` 응답의
   값을 다음 저장에 그대로 보낸다. `REVISION_MISMATCH`(ABORTED) 가 나면 다른
   곳에서 먼저 저장된 것 — `get_world_draft` 로 재조회한 뒤 편집을 다시 얹는다.
   저장 요청을 병렬로 보내지 말 것 — 반드시 앞 저장의 응답을 받고 다음을 보낸다.
5. **저장 성공이 불확실하면**(타임아웃 등) 같은 payload 를 재전송하지 말고
   `get_world_draft` 로 실제 상태를 확인한다 — `clientRef` 는 요청-응답 사이의
   상관 키일 뿐 멱등 키가 아니라, 재전송하면 엔티티가 중복 생성될 수 있다.
6. **PLAYER 캐릭터(`{{user}}`)는 건드리지 않는다** — 삭제·추가 불가, 항상 1명.
   서버가 만든 그대로 두고 NPC 만 추가한다.
7. 응답 draft 의 `coverMedia`·`tags`·캐릭터 `media[].media` 같은 리소스 상세는
   서버가 채우는 읽기 전용 필드다 — 저장 요청에 넣어도 무시된다. id 필드
   (`coverMediaId`·`tagIds`·`mediaId`)만 채우면 된다.

## 태그

- `list_tags(query)` 로 기존 태그를 먼저 검색하고, 없으면 `create_tag` 로
  만든다. **멱등**이라 중복 걱정 없이 바로 생성해도 된다.
- 태그 이름은 서버가 trim + 소문자로 정규화한다 — "Romance" 로 만들어도
  `"romance"` 로 저장·응답된다.
- 월드에 붙이는 건 `draft.info.tagIds` 로 — **전체 교체**라 기존 태그를
  유지하려면 그 id 도 함께 보낸다. 최대 5개.

## 게시 게이트

`validate_world_draft` 가 돌려주는 위반이 전부 해소돼야 게시된다. 필수 요건:

| 항목 | 요건 |
|---|---|
| 제목(32자)·소개(200자) | 필수 |
| 커버(`coverMediaId`) | 필수 — studio-media 스킬로 업로드 |
| 태그 | 1개 이상 |
| 공개범위(`visibility`) | **기본값 없음** — `WORLD_VISIBILITY_PUBLIC`/`PRIVATE` 중 반드시 선택 |
| 세계관 `core`·`style` | 필수 |
| NPC 캐릭터 | 1명 이상, 각각 `name`·`description` 필수 |
| 시나리오 | 1개 이상, 각각 `title`·`description`·아이템 1개 이상, 대사는 화자·내용 필수 |
| 장소 | (있다면) `description` 필수 · 사건은 `name`·장소 배정 필수 |

위반 응답의 `target`(어느 탭)·`entityId`·`field` 로 무엇이 비었는지 특정할 수
있다 — 사용자에게 "무엇을 채워야 게시되는지"를 이 목록으로 정리해 보여준다.

## 게시는 공개 행위다 — 반드시 확인받을 것

- `publish_world` 는 드래프트의 `visibility` 값 그대로 작품을 라이브에 올린다.
  **호출 전에 반드시 사용자에게 "지금 게시할까요? 공개범위는 X입니다" 를
  확인받는다.** validate 가 깨끗해도 자동으로 게시하지 않는다.
- 공개 → 비공개 전환도 재게시로만 가능하고, 재게시는 드래프트 전체 완성도를
  다시 요구한다. 공개 중 작품을 비공개로 내리는 재게시는 **플레이 중인 다른
  사용자가 즉시 진행 불가**가 된다 — 이 영향도 함께 알린다.
- `discard_world_draft` 는 미게시 변경을 버린다 — 한 번도 게시하지 않은
  월드에는 쓸 수 없다(버릴 게시본 기준이 없다). 되돌릴 수 없으므로 이것도
  사용자 확인 후 호출한다.
