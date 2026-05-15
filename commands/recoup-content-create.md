---
name: recoup-content-create
description: End-to-end run that produces a 9:16 social-ready short-form music video for an artist and song. The default front door for the plugin — invokes the short-video skill's async pipeline, polls until the video is rendered, and lands the user on the final clip + caption.
---

# Recoup Content Create

The anchor command. Use this on first install to confirm the plugin works end-to-end, and as the default front door for "make a video for [artist]" requests.

## What it does

1. Resolves the artist's `account_id` (from artist name + the user's first org).
2. Picks a template (defaults to `artist-caption-bedroom` if not supplied).
3. Fires `POST /api/content/create` to start the async pipeline server-side.
4. Polls `/api/tasks/runs?runId={runId}` every ~10 seconds until status is `COMPLETED`, `FAILED`, `CANCELED`, or `CRASHED`.
5. Reads the run output: `videoSourceUrl`, `imageUrl`, `captionText`, `template`, `lipsync`, and the audio metadata.
6. Lands the user on the final video URL with the caption text printed below it.

The async pipeline is the agent-safe path — synchronous calls to `/api/content/video` routinely take 60–180 seconds and time out inside most agent shells. See `skills/short-video/SKILL.md` for the underlying recipe and `references/short-video-manual.md` if you want to override a single stage.

## When to use the other commands

This command is the default. Skip it and use the underlying skills directly when:

- The user wants to **swap a single stage** (different caption length, different motion prompt, different reference image). Use the manual walkthrough in `references/short-video-manual.md`.
- The user wants to **generate one capability in isolation** (just an image, just a caption). Use the `content-creation` skill directly.

## Required environment

- `RECOUP_API_KEY` set in the shell environment, OR a Privy access token if running inside chat. The skill explains both auth modes.
- An artist record already exists for the requested artist on the user's first org.
- For lipsync templates: a `song.mp3` resolvable from the artist's sandbox repo, YouTube, or a path the user supplies (see `references/song-sourcing.md`).

## What "complete" looks like

The command finishes when:

- The poll loop has seen a terminal status (`COMPLETED` is the happy path).
- `output.videoSourceUrl` is a fetchable URL.
- `output.captionText` is a non-empty string.

Print both to the user. If status is `FAILED`, `CANCELED`, or `CRASHED`, surface the error from `runs[0].error` and stop — don't claim success.

## Steps the agent must execute

Follow `skills/short-video/SKILL.md`'s **async pipeline** section. The skill description triggers automatically on this command's intent ("create content", "make a video"), so the agent should pick it up via skill matching. If it doesn't, name it explicitly:

> Use the `short-video` skill to run the async pipeline for `$ARTIST_NAME` with template `$TEMPLATE`.

Default template: `artist-caption-bedroom`. Override by asking the user, or by detecting cues like "with the album cover" → `album-record-store`.
