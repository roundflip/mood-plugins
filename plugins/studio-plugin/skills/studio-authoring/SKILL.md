---
name: studio-authoring
description: mood 스튜디오에서 세계관(월드)을 만들고 편집·게시할 때 사용. 사용자와 대화로 세계관을 구체화한 뒤 최종 컨펌을 받고 나서야 등록하는 프로세스와, mood-studio MCP 도구(create_world, get/save_world_draft, validate_world_draft, publish_world, 태그)의 저장·게시 규약을 다룬다. "mood에 작품/세계관을 만들어줘", "드래프트 수정", "게시해줘"라고 하면 이 스킬을 따른다.
---

# mood 스튜디오 저작 — 대화로 구체화하고, 컨펌 후 등록한다

<HARD-GATE>
사용자가 최종 설계를 승인하기 전에는 어떤 쓰기 도구도 호출하지 않는다 —
create_world 로 빈 월드를 만드는 것도 포함이다. 탐색·설계 단계에서 허용되는
도구는 읽기(list_my_worlds, get_world_draft, list_tags)뿐이다. 게시(publish_world)와
드래프트 폐기(discard_world_draft)는 등록 승인과 별개로 그 시점에 다시 확인받는다.
</HARD-GATE>

## 인증

저작 도구가 안 보이고 `authenticate` 도구만 있다면 미인증 상태다 —
`authenticate` 를 호출해 받은 인가 URL 을 사용자에게 브라우저로 열도록
안내하고, 사용자가 로그인·동의를 마쳐 도구 11종이 노출되면 진행한다.
콜백 페이지가 연결 오류로 뜨면 사용자에게 주소창의 URL 전체를 받아
`complete_authentication` 에 그대로 넘긴다. 인증은 언제 일어나도 되지만,
브레인스토밍 도중 대화를 끊지 않도록 **등록 직전(4단계 진입 시)** 에
확인하는 것을 권장한다. 인증 완료 직후에는 도구 목록 갱신이 한 턴 늦을 수
있다 — 저작 도구가 아직 안 보여도 실패로 판단하지 말고 한 턴 뒤에 다시
확인한다.

## 프로세스

먼저 요청을 분류하고 소리 내어 말한다 — "새 세계관이니 구체화부터 할게요" /
"기존 작품의 작은 수정이니 변경안만 확인받고 저장할게요":

- **신규 세계관 / 대규모 개편** → 아래 4단계 전체를 따른다.
- **소수정** (오타, 문구 다듬기, 태그 교체 등 기존 작품의 국소 변경) →
  `get_world_draft` 로 현재 상태를 읽고, 바꿀 내용을 요약해 보여준 뒤 승인받고
  저장한다. 브레인스토밍은 생략하되 **승인 게이트는 생략하지 않는다.**
- 애매하면 무거운 쪽(신규/대개편 프로세스)을 택한다. 작업 도중 변경 범위가
  커지면 멈추고 말한 뒤 단계를 올린다.

### 1. 탐색 — 질문은 한 번에 하나씩

사용자의 머릿속 아이디어를 끌어내는 단계다. 심문이 아니라 대화가 되도록
**한 메시지에 질문 하나**만 던진다. 선택지를 제시할 수 있으면 객관식이 좋다.
이미 답이 나온 것을 다시 묻지 않는다. 다뤄야 할 것:

- 장르·배경 (현대/판타지/SF..., 어떤 세계인가)
- `{{user}}`(플레이어)가 맡는 역할과 처지
- 핵심 NPC — 누구와 어떤 관계를 쌓는 이야기인가
- 긴장의 원천 — 이 세계에서 무엇이 이야기를 굴리는가
- 톤과 문체 (수위 포함)
- 시작 장면 — 플레이가 열리는 순간

사용자가 이미 구체적인 설정을 들고 왔다면 빈 곳만 묻는다. 각 필드를 어떻게
쓰는 게 좋은지는 studio-writing 스킬의 작법 기준을 따른다.

### 2. 설계 제시 — 섹션별로 확인

모은 재료로 세계관 설계를 **섹션 단위로** 제시하고, 섹션마다 괜찮은지 확인받는다
(한 번에 전부 쏟아내지 않는다):

1. 세계관 — 제목 후보·한 줄 소개·core(배경/전제)·style(문체)·rules
2. 캐릭터 — NPC 각각의 이름·description·personality·example_line·relationship
3. 시작 시나리오 — 제목·소개·인트로 장면(items 초안)·선택지

피드백이 나오면 반영해 그 섹션을 다시 보여준다. 사용자가 "알아서 해줘"라고
해도 최종 요약은 반드시 거친다.

### 3. 최종 컨펌 — 등록 게이트

전체 설계를 한 번에 요약해 보여주고 명시적으로 묻는다:
**"이대로 mood 에 등록할까요? (등록 = 내 작품에 드래프트 저장, 게시 전이라
다른 사람에게는 보이지 않습니다)"**

승인받기 전에는 쓰기 도구를 호출하지 않는다. "간단하니 만들면서 다듬자"는
생각이 들면 그게 게이트를 건너뛰려는 신호다 — 건너뛰지 않는다.

### 4. 등록 — 이때부터 도구를 쓴다

```
create_world                              → world_id
get_world_draft(world_id)                 → { draft, revision }  (PLAYER 캐릭터 포함 초안)
(설계를 draft 에 반영)
save_world_draft(world_id, draft, revision)
list_tags / create_tag → tagIds 반영해 재저장
validate_world_draft(world_id)            → 남은 위반 정리해 보고
```

등록이 끝나면 결과(작품 id·저장된 구성·validate 위반 목록)를 보고하고, 다음
단계를 안내한다 — 커버 이미지는 studio-media 스킬, 게시는 아래 게이트.

## 드래프트 스키마 — save_world_draft 의 `draft` 형태

`get_world_draft` 응답의 draft 를 기반으로 수정해 보낸다. **저장은 전체 스냅샷
교체다** — 일부 섹션만 보내면 나머지(빠뜨린 캐릭터·시나리오 등)가 삭제된다.
최상위 배열 4종(places·events·characters·scenarios)은 비어 있어도 `[]` 로 보낸다.

```
info: { title, description, core, style,
        rules: [{ name, content }],          # 문자열 배열 아님 — name·content 쌍
        endingConditions: [string],
        tagIds: [string], coverMediaId?,     # id 는 전부 JSON 문자열
        targetGender, visibility }           # enum 문자열, visibility 는 기본값 없음
places: [{ clientRef|id, name, description }]
events: [{ clientRef|id, name, trigger, progression,
           placeId|placeClientRef? }]        # 장소는 선택
characters: [{ clientRef|id, name, description, personality,
               exampleLine, relationship, profileMediaId?,
               media: [{ clientRef|id, mediaId, description,
                         unlockRequired }] }]
               # type 은 서버가 정한다(PLAYER/NPC) — 보내는 필드가 아님
scenarios: [{ clientRef|id, title, description,
              items: [ { narration: { content } }
                     | { speech: { characterClientRef|characterId, content } }
                     | { effect: { style } }
                     | { media: { mediaId } } ],
              choices: [string] }]           # 최대 3개 · 각 60자
```

대사·나레이션 본문 필드는 `text` 가 아니라 **`content`** 다. 응답에만 있는
읽기 전용 필드(`coverMedia`·`tags`·`media[].media`·`profileMedia`·캐릭터
`type`)는 저장 요청에 넣어도 무시된다 — 규약 7 참조.

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
   응답에 `revision` 필드가 없으면 값이 **0** 인 것이다(0 은 JSON 에서 생략된다)
   — 첫 저장에는 0 을 보낸다.
5. **저장 성공이 불확실하면**(타임아웃 등) 같은 payload 를 재전송하지 말고
   `get_world_draft` 로 실제 상태를 확인한다 — `clientRef` 는 요청-응답 사이의
   상관 키일 뿐 멱등 키가 아니라, 재전송하면 엔티티가 중복 생성될 수 있다.
6. **PLAYER 캐릭터(`{{user}}`)는 건드리지 않는다** — 삭제·추가 불가, 항상 1명.
   서버가 만든 그대로 두고 NPC 만 추가한다.
7. 응답 draft 의 `coverMedia`·`tags`·캐릭터 `media[].media` 같은 리소스 상세는
   서버가 채우는 읽기 전용 필드다 — 저장 요청에 넣어도 무시된다. id 필드
   (`coverMediaId`·`tagIds`·`mediaId`)만 채우면 된다.
8. **저장은 관대하고, 게시는 엄격하다.** 빈 값·미완성은 저장이 받아준다. 단
   **존재하지 않는 대상을 가리키는 참조**(없는 장소 id, 남의 미디어 id 등)는
   저장에서도 거부된다.
9. **쓰기 도구는 한 응답에 하나만 호출한다.** 같은 모양의 짧은 쓰기 호출을
   한 응답에 병렬로 나열하면 동일 호출이 통제 없이 반복되는 사고가 실제로
   있었다(2026-08-20, create_tag 135회 중복 — 멱등이라 무사했지만
   save_world_draft 였다면 엔티티 중복 생성이다). 읽기는 병렬로 해도 되지만,
   쓰기는 앞 호출의 응답을 확인하고 다음을 보낸다.

## 태그

- `list_tags(query)` 로 기존 태그를 먼저 확인하고, 없는 것만 `create_tag` 로
  만든다. **멱등**이라 중복 걱정은 없지만, 규약 9에 따라 **한 응답에 하나씩
  순차로** 호출한다 — 여러 개를 병렬로 나열하지 않는다.
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

## 게시는 공개 행위다 — 별도로 다시 확인받는다

- `publish_world` 는 드래프트의 `visibility` 값 그대로 작품을 라이브에 올린다.
  등록 승인과는 **별개의 결정**이다 — 호출 전에 반드시 "지금 게시할까요?
  공개범위는 X입니다"를 확인받는다. validate 가 깨끗해도 자동으로 게시하지
  않는다.
- 공개 → 비공개 전환도 재게시로만 가능하고, 재게시는 드래프트 전체 완성도를
  다시 요구한다. 공개 중 작품을 비공개로 내리는 재게시는 **플레이 중인 다른
  사용자가 즉시 진행 불가**가 된다 — 이 영향도 함께 알린다.
- `discard_world_draft` 는 미게시 변경을 버린다 — 한 번도 게시하지 않은
  월드에는 쓸 수 없다. 되돌릴 수 없으므로 이것도 확인 후 호출한다.

## 레드 플래그

| 생각 | 실제 |
|---|---|
| "간단한 세계관이니 바로 만들자" | 간단하면 질문이 줄어들 뿐, 컨펌 게이트는 줄지 않는다 |
| "빈 월드만 먼저 만들어 두자" | create_world 도 쓰기다 — 승인 전 호출 금지 |
| "사용자가 알아서 해달라고 했다" | 위임은 설계를 맡긴 것이지 확인을 생략하라는 뜻이 아니다 — 최종 요약은 보여준다 |
| "validate 가 깨끗하니 게시까지 하자" | 게시는 별도 확인이 필요한 공개 행위다 |
