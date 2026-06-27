---
description: One-time setup — allow this plugin's tools so you aren't prompted each message
---

Claude Code does not let a plugin ship its own permission rules, so this command
adds them to the user's settings on request.

Goal: allow all of this plugin's MCP tools (`reply`, `ac2_sign`, `ac2_verify_*`,
`ac2_capabilities`, …) without a per-call permission prompt.

Do this:

1. Read `~/.claude/settings.json` (create it as `{}` if it doesn't exist).
2. Ensure `permissions.allow` is an array and contains the exact string
   `mcp__plugin_ac2_ac2-channel__*`. Add it if missing; do NOT remove or reorder any
   existing entries. (That wildcard token is `mcp__plugin_<pluginName>_<serverKey>__*`
   for this plugin — plugin name `ac2`, server key `channel`.)
3. Write the file back as valid JSON.
4. Tell the user it's done and that they should restart Claude Code (or run
   `/reload-plugins`) for it to take effect.

If the user later "always allows" a tool from the prompt, Claude Code writes the
canonical rule to `~/.claude/settings.local.json` — that's the source of truth
for the exact token if anything here looks off.
