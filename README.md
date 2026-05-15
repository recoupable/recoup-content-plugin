# Recoup Content Plugin

Agent plugin for content workflows on the
[Recoup](https://recoupable.com) platform. A home for skills, commands,
and helpers that let AI agents draft, edit, and publish content for
artists — captions, blurbs, release copy, social posts, and anything
else that lives on Recoup's content surfaces.

This plugin is a **starter skeleton**: manifests are in place so the
marketplace can register the repo, but skills, agents, and commands
will arrive in follow-up releases.

## Install

### Claude Code (CLI)

```bash
claude plugin install https://github.com/recoupable/recoup-content-plugin
```

### Claude Cowork

1. Open the plugin marketplace (puzzle-piece icon in the sidebar).
2. Click **Add custom plugin** and paste:
   `https://github.com/recoupable/recoup-content-plugin`
3. Approve the requested tool permissions.
4. Restart the Cowork session so manifests load.

### Cursor

1. Cursor → Settings → Plugins → **Add custom plugin**.
2. Paste the GitHub URL above.
3. Restart Cursor so `.cursor-plugin/plugin.json` loads.

## Layout

```
.claude-plugin/plugin.json   # Claude Code manifest
.codex-plugin/plugin.json    # Codex manifest
.cursor-plugin/plugin.json   # Cursor manifest
skills/                      # (to be added) content workflow skills
agents/                      # (to be added) content personas
commands/                    # (to be added) slash commands
```

## Roadmap

- Drafting skills for captions, release blurbs, and social posts.
- Editing skills that pick up artist brand voice from the platform.
- Publishing commands that push content through Recoup's surfaces.

## Support

- Email: `support@recoupable.com`
- Website: <https://recoupable.com>

## License

[Apache-2.0](./LICENSE)
