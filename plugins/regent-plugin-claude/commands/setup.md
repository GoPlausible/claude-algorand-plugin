---
description: One-time setup — enable the Regent channel + skip per-tool permission prompts
---

Claude Code doesn't let a plugin ship its own permission or channel settings, so
this command writes them into the user's settings on request.

Goals: (a) run the Regent channel without `--dangerously-load-development-channels`,
and (b) allow this plugin's MCP tools (`reply`, `regent_sign`, `regent_verify_*`,
`regent_capabilities`, …) without a per-call permission prompt.

Edit `~/.claude/settings.json` (create it as `{}` if it doesn't exist), making
ALL of the following changes. Preserve every existing key/entry — only add what's
missing, never remove or reorder. Then write the file back as valid JSON.

1. **Permissions** — ensure `permissions.allow` is an array containing the exact
   string `mcp__plugin_regent_regent-channel__*` (add if missing). That wildcard token
   is `mcp__plugin_<pluginName>_<serverKey>__*` — plugin `regent`, server key
   `regent-channel`.

2. **Channel master switch** — set `"channelsEnabled": true`.

3. **Channel allowlist** — ensure `allowedChannelPlugins` is an array that
   contains this object (add it if no entry already has the same `marketplace`
   AND `plugin`):

   ```json
   { "marketplace": "goplausible-claude-plugins", "plugin": "regent" }
   ```

After writing, tell the user to restart Claude Code (or `/reload-plugins`), then
launch with `claude --channels plugin:regent@goplausible-claude-plugins` — no
`--dangerously-load-development-channels` needed.

Notes:
- If the user "always allows" a tool from a prompt, Claude Code writes the
  canonical permission rule to `~/.claude/settings.local.json` — the source of
  truth for the exact token if anything here looks off.
- `allowedChannelPlugins` is documented as a managed/admin setting but is honored
  in a personal user `settings.json`.
