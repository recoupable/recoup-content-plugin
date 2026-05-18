---
name: recoup-content-create
description: End-to-end async run that produces a 9:16 social-ready short-form music video for an artist and song. Use whenever the user types `/recoup-content-create`, says "make a video for [artist]", "create a TikTok for [artist]", "produce a Reel for [artist]", "kick off content for [artist]", or any front-door request to generate finished social-ready content for an existing artist. Resolves the artist's `account_id`, fires `POST /api/content/create`, polls `/api/tasks/runs` until terminal, and lands the user on the final video URL + caption. The default front door for the recoup-content-plugin.
argument-hint: <artist-name> [--template <template-name>]
allowed-tools: [Bash, Read, Write, AskUserQuestion]
---

# Recoup Content Create

The anchor command for the recoup-content-plugin. Use this on first install to confirm the plugin works end-to-end, and as the default front door for "make a video for [artist]" requests.

This skill is **user-invoked** — the user types `/recoup-content-create <artist-name>` and the harness invokes it with `$ARGUMENTS` populated. The model-invoked skills (`short-video`, `content-creation`) are what this skill chains to under the hood.

## Arguments

The user invoked this with: `$ARGUMENTS`

Expected shape: `<artist-name> [--template <template-name>]`

- **`<artist-name>`** — the artist to generate content for. Required. If the user didn't supply one, AskUserQuestion to collect it.
- **`--template <name>`** — optional. Defaults to `artist-caption-bedroom`. Override based on user cue ("with the album cover" → `album-record-store`, etc.).

## What it does

1. Resolves the artist's `account_id` (from artist name + the user's first org).
2. Picks a template (defaults to `artist-caption-bedroom` if not supplied).
3. Fires `POST /api/content/create` to start the async pipeline server-side.
4. Polls `/api/tasks/runs?runId={runId}` every ~10 seconds until status is `COMPLETED`, `FAILED`, `CANCELED`, or `CRASHED`.
5. Reads the run output: `videoSourceUrl`, `imageUrl`, `captionText`, `template`, `lipsync`, and the audio metadata.
6. Lands the user on the final video URL with the caption text printed below it.

The async pipeline is the agent-safe path — synchronous calls to `/api/content/video` routinely take 60–180 seconds and time out inside most agent shells. See `skills/short-video/SKILL.md` for the underlying recipe and `references/short-video-manual.md` if you want to override a single stage.

## When to use the other skills

This is the default. Skip it and use the underlying skills directly when:

- The user wants to **swap a single stage** (different caption length, different motion prompt, different reference image). Use the manual walkthrough in `references/short-video-manual.md`.
- The user wants to **generate one capability in isolation** (just an image, just a caption). Use the `content-creation` skill directly.

## Required environment

- `RECOUP_API_KEY` set in the shell environment, OR a Privy access token if running inside chat. The `short-video` skill explains both auth modes.
- An artist record already exists for the requested artist on the user's first org. If not, point the user at `recoup-platform-plugin`'s `create-artist` skill first.
- For lipsync templates: a `song.mp3` resolvable from the artist's sandbox repo, YouTube, or a path the user supplies (see `references/song-sourcing.md`).

## What "complete" looks like

The command finishes when:

- The poll loop has seen a terminal status (`COMPLETED` is the happy path).
- `output.videoSourceUrl` is a fetchable URL.
- `output.captionText` is a non-empty string.

Print both to the user. If status is `FAILED`, `CANCELED`, or `CRASHED`, surface the error from `runs[0].error` and stop — don't claim success.

## Steps the agent must execute

Follow `skills/short-video/SKILL.md`'s **async pipeline** section. The skill description triggers automatically on this command's intent, so the agent should pick it up via skill matching. If it doesn't, name it explicitly:

> Use the `short-video` skill to run the async pipeline for `$ARTIST_NAME` with template `$TEMPLATE`.

Default template: `artist-caption-bedroom`. Override by asking the user, or by detecting cues like "with the album cover" → `album-record-store`.
