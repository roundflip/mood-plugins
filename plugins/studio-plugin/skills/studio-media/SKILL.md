---
name: studio-media
description: mood 스튜디오 작품에 이미지(커버·캐릭터 이미지·컷·배경)를 올릴 때 사용. request_media_upload → presigned POST 직접 업로드 → complete_media_upload 2단계 흐름과 usage 자리 규칙을 다룬다. 사용자가 "커버 올려줘", "캐릭터 이미지 추가"라고 하면 이 스킬을 따른다.
---

# mood 스튜디오 미디어 업로드

이미지 바이너리는 MCP 도구를 **통과하지 않는다** — 도구는 업로드 주소를
발급하고 확정만 한다. 실제 바이트는 발급받은 presigned URL 로 직접 POST 한다.

## 3단계 흐름

```
1. request_media_upload(object_type_name, filename, content_type)
     → { uploadUrl, formFields[], fileFieldName, objectStorageKey, maxContentLength }
2. uploadUrl 로 multipart/form-data POST  (아래 § 직접 업로드)
3. complete_media_upload(objectStorageKey, usage, image_object_id)
     → { mediaId }   ← 이 id 를 드래프트의 해당 자리에 넣는다
```

- `object_type_name` 은 `studio-gallery-image` 를 쓴다.
- `image_object_id` 는 1단계 응답 `formFields` 중 `x-amz-meta-object-id` 값이다.
- presigned URL 은 **만료 시간이 있다**(`expireTime`) — 발급 후 곧바로 업로드한다.

## 직접 업로드 (curl)

`formFields` 의 **모든 필드를 이름 그대로**, 마지막에 파일 필드(`fileFieldName`,
보통 `file`) 순서로 보낸다. 필드를 하나라도 빠뜨리거나 값을 바꾸면 정책 서명이
깨져 403 이 난다.

```bash
curl -s -w "%{http_code}" -X POST "{uploadUrl}" \
  --form-string "Content-Type=image/png" \
  --form-string "key=..." \
  --form-string "policy=..." \
  --form-string "tagging=<Tagging>...</Tagging>" \
  ...(formFields 전부)... \
  -F "file=@cover.png;type=image/png"
```

⚠️ **curl 은 반드시 `--form-string` 을 쓴다** (`-F` 아님) — `tagging` 값이
`<` 로 시작해서 `-F` 는 이를 파일 경로로 오해해 read error 로 죽는다. 파일
필드만 `-F "file=@경로"` 를 쓴다. 성공은 **204** 다.

## usage — 자리마다 정해진 용도

`complete_media_upload` 의 `usage` 와 드래프트에서 그 media 를 넣는 자리가
**정확히 일치**해야 한다. 어긋나면 드래프트 저장이 `INVALID_REFERENCE` 로
거부된다:

| usage | 드래프트의 자리 |
|---|---|
| `MEDIA_USAGE_COVER` | `info.coverMediaId` |
| `MEDIA_USAGE_CHARACTER_PROFILE` | `characters[].profileMediaId` (대표 이미지) |
| `MEDIA_USAGE_CHARACTER_CUT` | `characters[].media[].mediaId` (갤러리 컷 — 플레이로 해금) |
| `MEDIA_USAGE_SCENARIO_BACKGROUND` | 시나리오 media 아이템의 `mediaId` |

- 같은 media 를 usage 가 다른 두 자리에 동시에 쓸 수 없다 — 자리마다 그 usage 로
  새로 만든다.
- 대표 이미지(profile)와 컷(cut)은 **별개 media** 다 — 컷을 대표 자리에 꽂을 수
  없다.
- **컷은 캐릭터 간 이동 금지** — 같은 이미지를 다른 캐릭터에도 쓰려면 기존 컷을
  옮기지 말고 같은 `mediaId` 를 참조하는 새 컷 항목을 추가한다.
- media 는 본인 소유만 참조 가능 — 타인/미존재 id 는 거부된다.

## 재시도

`complete_media_upload` 가 두 단계(업로드 확정 → media 생성) 중 media 생성만
실패했다면 같은 인자로 재호출한다(업로드 확정은 멱등). 단 **성공 응답을 이미
받았는데 또 호출하면 media 가 중복 생성**되니, 성공 여부가 불확실할 때만
재시도한다.
