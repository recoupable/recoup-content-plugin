---
name: short-video
description: End-to-end playbook for producing a vertical, social-ready short-form music video (image + motion + caption + composed final) from an artist and a song. Use when the user asks to make a video, generate content for an artist, create a TikTok or Reel, produce a music-video clip, or kick off the content pipeline — phrases like "make a video about", "generate content for", "create a TikTok", "kick off a video for X", "short-form video", or "music video for $ARTIST". The skill has two paths: an async pipeline (`POST /api/content/create` + poll `/api/tasks/runs`) for short-shell agents (preferred), and a manual five-step recipe in `references/short-video-manual.md` when you want to swap a single stage.
---

# Short Video

The canonical recipe used internally by Recoup's `create-content` background task. Produces a 9:16 music-video clip from an existing artist + song, ready to post.

## Two paths

| Path | When to use | Where it lives |
|---|---|---|
| **Async pipeline** | Default for agents — fires server-side and polls. Survives short shell timeouts. | This file (below). |
| **Manual walkthrough** | Humans or long-lived shells that want explicit control over a single step (different caption length, different motion, different audio). | `references/short-video-manual.md` |

For sourcing the underlying `song.mp3`, see `references/song-sourcing.md` — it covers the sandbox-repo → YouTube → user-supplied fallback chain. **Do not silently default to a placeholder track.**

## Async pipeline (preferred for agents)

`POST /api/content/video` is synchronous; provider latency can exceed a host command timeout. Prefer the async hosted workflow in short-lived shells, retain its run ID, and read its status in a later command if necessary.

```bash
BASE="https://api.recoupable.dev/api"
CREATED=$(curl --fail-with-body -sS -X POST "$BASE/content/create" -H "x-api-key: $RECOUP_API_KEY" -H "Content-Type: application/json" \
  -d "$(jq -n --arg a "$ARTIST_ACCOUNT_ID" --arg t "$TEMPLATE" '{artist_account_id:$a, template:$t}')") || exit 1
RUN_ID=$(printf '%s' "$CREATED" | jq -er '.runIds[0]') || exit 1
printf 'Run ID: %s\n' "$RUN_ID"

for ATTEMPT in $(seq 1 60); do
  RUN=$(curl --fail-with-body -sS -H "x-api-key: $RECOUP_API_KEY" "$BASE/tasks/runs?runId=$RUN_ID") || exit 1
  STATUS=$(printf '%s' "$RUN" | jq -er '.runs[0].status') || exit 1
  case "$STATUS" in
    COMPLETED) break ;;
    FAILED|CANCELED|CRASHED|SYSTEM_FAILURE|INTERRUPTED)
      printf '%s' "$RUN" | jq '.runs[0].error // {status: .runs[0].status}' >&2
      exit 1 ;;
  esac
  sleep 10
done
[ "$STATUS" = COMPLETED ] || { echo "Still pending: $RUN_ID"; exit 2; }

# Read the output
curl -sS "https://api.recoupable.dev/api/tasks/runs?runId=$RUN_ID" \
  -H "x-api-key: $RECOUP_API_KEY" \
  | jq '.runs[0].output'
# -> { videoSourceUrl, imageUrl, captionText, template, lipsync, audio: {...} }
```

See [Tasks Runs](https://docs.recoupable.dev/api-reference/tasks/runs) for the full status enum and the `CreateContentRunOutput` schema.

### Resolving `$ARTIST_ACCOUNT_ID`

`POST /api/content/create` needs the artist's `account_id`. Use the connected
account and a user-selected organization; never assume the first organization is
correct or create a new account because a roster is empty. OAuth MCP credentials
do not automatically create a REST shell key. Obtain an authorized REST credential
through the platform setup flow when using these curl examples.

List organizations, inspect the returned array of organization memberships and
select the intended `organization_id` before setting `ORG_ID`. The response
entry `id` is a membership-row identifier and must not be used as `ORG_ID`. Then
list the roster:

```bash
curl --fail-with-body -sS "https://api.recoupable.dev/api/organizations" \
  -H "x-api-key: $RECOUP_API_KEY"
# Set ORG_ID to the explicitly selected organization UUID.
curl --fail-with-body -sS "https://api.recoupable.dev/api/artists?org_id=$ORG_ID" \
  -H "x-api-key: $RECOUP_API_KEY"
# Select the intended artist and set ARTIST_ACCOUNT_ID to its account_id.
```

The artist record exposes both `id` and `account_id` (both UUIDs). Use **`account_id`** — `id` is the artist row's primary key, `account_id` is the underlying account that owns it. Do not substitute the artist row ID for the required account ID.

## When the agent should reach for the manual walkthrough

The async pipeline produces a finished video with sensible defaults. Switch to `references/short-video-manual.md` when:

- The user wants a **different caption length** than the template default.
- The user supplies their own **reference image** and wants to skip generation.
- The user wants to **swap the audio** with a specific clip rather than the song from the sandbox.
- The user wants to **inspect intermediate outputs** (the raw image, the un-composed 16:9 clip) before composition.

The manual walkthrough is the same five steps, run individually. It is significantly slower in short shells — use it only when the async pipeline's defaults don't fit.
