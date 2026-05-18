# Recoup Content Plugin

Agent plugin for content workflows on the [Recoup](https://recoupable.com) platform. Lets AI agents draft, edit, and publish content for artists — short-form music videos, captions, images, and the supporting building blocks.

Built around the `/api/content/*` endpoints and the `recoup content` CLI. Driven by a single front-door command — `/recoup-content-create` — that produces a finished 9:16 social-ready clip from an artist + song.

## Install

### Claude Code (CLI)

```bash
claude plugin install https://github.com/recoupable/recoup-content-plugin
```

### Claude Cowork

1. Open the plugin marketplace (puzzle-piece icon in the sidebar).
2. Click **Add custom plugin** and paste:
   `https://github.com/recoupable/recoup-content-plugin`
3. Approve the requested tool permissions (`Read`, `Write` — needed to write the workspace and final `.mp4`).
4. Restart the Cowork session so manifests load.

### Cursor

1. Cursor → Settings → Plugins → **Add custom plugin**.
2. Paste the GitHub URL above.
3. Restart Cursor so `.cursor-plugin/plugin.json` loads commands and skills.

## Getting started

After install, set your Recoup API key in the shell:

```bash
export RECOUP_API_KEY="recoup_sk_..."   # see https://developers.recoupable.com/agents
```

Then in a new chat:

> **Make a TikTok for Mari Vega**

Or invoke the anchor command directly:

```text
/recoup-content-create
```

The agent picks up the `short-video` skill, resolves the artist's `account_id`, fires the async pipeline, polls until the render finishes, and lands you on the final video URL + caption.

## Commands

| Command | When to use it |
| ------- | -------------- |
| `/recoup-content-create` | **Default.** End-to-end async run from artist name to finished video + caption. The front door for first installs and "make me a video" requests. |

> **v0.3.0 spec migration:** `/recoup-content-create` was migrated from `commands/recoup-content-create.md` to `skills/recoup-content-create/SKILL.md` per Anthropic's newer skills-not-commands convention (the official `claude-plugins-official` example-plugin declares the `commands/*.md` layout legacy). Both files exist in this release for back-compat; the legacy command file will be removed in a future version.

More command coverage (templates browse, single-step overrides, demo workspace) is on the roadmap.

## Skills

Loaded automatically by description-matching when the agent recognizes the task:

| Skill | What it does |
| ----- | ------------ |
| `short-video` | End-to-end playbook for producing a 9:16 music video. Two paths — async pipeline (preferred for agents) and a manual five-step recipe externalized to `references/`. |
| `content-creation` | Atomic primitives for the `/api/content/*` surface — generate an image, generate a video (6 modes including lipsync), generate a caption, transcribe audio, edit, upscale, analyze. Use when you want one capability in isolation rather than the full recipe. |

`short-video` is the **recipe**; `content-creation` is the **building blocks**. Pick the recipe when the user wants a finished deliverable, pick the building blocks when they want fine-grained control.

## References

Long-form prose lives in `references/` so `SKILL.md` files stay scannable for the model:

- `references/song-sourcing.md` — sandbox-repo → YouTube → user-supplied fallback chain for resolving the underlying `song.mp3`.
- `references/short-video-manual.md` — full five-step manual recipe for humans or long-lived shells that want explicit control over a single stage.

## Required environment

- `RECOUP_API_KEY` (`recoup_sk_…`) — obtain via `POST /api/agents/signup`. See [Agents](https://developers.recoupable.com/agents).
- For the manual walkthrough's local compose step: `ffmpeg` on PATH.

## Structure

```text
recoup-content-plugin/
├── .claude-plugin/plugin.json
├── .codex-plugin/plugin.json
├── .cursor-plugin/plugin.json
├── commands/
│   └── recoup-content-create.md      # anchor front-door command
├── skills/
│   ├── content-creation/SKILL.md     # atomic /api/content/* primitives
│   └── short-video/SKILL.md          # async recipe (manual recipe in references/)
├── references/
│   ├── song-sourcing.md              # song.mp3 fallback chain
│   └── short-video-manual.md         # full step-by-step manual recipe
├── LICENSE
└── README.md
```

## Roadmap

- `/recoup-content-templates` — browse and inspect content templates without running a full pipeline.
- `/recoup-content-demo` — bundled fixture so first-install users see output before pointing the plugin at a real account.
- `hooks/` — a `SessionStart` hook that validates `RECOUP_API_KEY` and `ffmpeg` presence, and a `Stop` hook that prevents the agent from claiming "video ready" without a non-zero-byte `.mp4`.
- More skills covering artist-brand-voice editing and Recoup-surface publishing.

## Support

- Email: `support@recoupable.com`
- Website: <https://recoupable.com>

## License

[Apache-2.0](./LICENSE)
