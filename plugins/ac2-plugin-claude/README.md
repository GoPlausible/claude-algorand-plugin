# @goplausible/ac2-plugin-claude

AC2 channel plugin for **Claude Code**. Pairs Claude Code with an AC2 wallet
(e.g. [Regent](https://liquidauth.goplausible.xyz/download/regent.apk)) over
Liquid Auth (FIDO2) + WebRTC DataChannels, giving you:

- **Two-way chat** — message Claude Code from your wallet; replies come back.
- **Cryptographic signing** — `ac2_sign` asks the wallet to sign bytes (the user
  approves in-wallet with passkey/biometric); `ac2_verify_*` verify signatures.
- **Capability discovery** — `ac2_capabilities` enumerates the wallet's
  identities and per-chain accounts.
- **Remote permission approval (HITL)** — Claude Code tool-approval prompts
  (Bash/Write/Edit) are relayed into your wallet chat; reply `yes <id>` / `no <id>`
  to approve or deny from your phone.

This is a **Claude Code channel** — an MCP server (stdio) that pushes events into
a running Claude Code session. It reuses the entire AC2 core from
[`@goplausible/ac2-plugin-openclaw`](../ac2-plugin-openclaw) (the same
SessionManager, signing, capabilities, and auto-reconnect), wrapped behind the
[Claude Code channel contract](https://code.claude.com/docs/en/channels-reference)
instead of OpenClaw's.

## Requirements

- **Claude Code v2.1.80+** (v2.1.81+ for permission relay) on a claude.ai
  subscription (channels don't work with API-key auth).
- **Node 20+** (the runtime — `@roamhq/wrtc` is a native addon that loads in
  Node, not Bun/Deno).

## Build model (standalone bundle)

`npm run build` runs [`build.mjs`](./build.mjs), which esbuilds `server.ts` + all
of `src/` **and every pure-JS dependency** into a single self-contained
`dist/server.js`. This means the built server does **not** depend on the
monorepo's hoisted root `node_modules` (or any workspace layout) at runtime.

The **one** runtime dependency is [`@roamhq/wrtc`](https://www.npmjs.com/package/@roamhq/wrtc)
— a native addon (ships a `.node` binary) that can't be inlined into JS, so it's
left external and loaded at runtime via `createRequire`. Its per-platform
binaries are `optionalDependencies`. Everything else is a `devDependency`
(needed only to build the bundle). Plugin/SDK version strings are injected at
build time (esbuild `define`), so the bundle has no runtime `package.json` read.

## Local development

Channels are a research preview; a custom channel needs the development flag.

```bash
# 1. Install + build (from this directory)
npm install
npm run build   # → standalone dist/server.js (esbuild)

# 2. Load the plugin for the session and enable its channel. --plugin-dir loads
#    the /ac2:* slash commands AND the MCP server (from this package's .mcp.json,
#    via ${CLAUDE_PLUGIN_ROOT} — no hand-written config needed).
claude --plugin-dir /abs/path/to/packages/ac2-plugin-claude \
       --dangerously-load-development-channels server:plugin:ac2:ac2-channel
```

The dev-channel flag takes the MCP **server id**, which for a plugin-provided
server is `plugin:<pluginName>:<serverKey>` → here `plugin:ac2:ac2-channel`
(plugin name `ac2`, server key `ac2-channel` from `.mcp.json`). If the id is wrong,
the server still connects and tools work, but the logs show
`Channel notifications skipped: server <id> not in --channels list` and inbound
chat is dropped. Confirm the exact id with `/mcp` or the stderr logs.

Then in the session: run `/ac2:pair` (or ask "pair my wallet") — it shows a QR +
deep link, scan it with Regent, and you're connected. Slash commands:
`/ac2:pair`, `/ac2:status`, `/ac2:capabilities`, `/ac2:sign`, `/ac2:version`,
`/ac2:forget`, `/ac2:help`.

## Permissions (avoid a prompt on every message)

Claude calls this plugin's tools (`reply`, `ac2_sign`, `ac2_verify_*`,
`ac2_capabilities`, …) constantly, so you'll get a permission prompt each time
unless you pre-approve them. Do **one** of:

- **Allow-list the plugin's tools** (recommended) — add this to your
  `~/.claude/settings.json` (or project `.claude/settings.json`) under
  `permissions.allow`:

  ```json
  { "permissions": { "allow": ["mcp__plugin_ac2_ac2-channel__*"] } }
  ```

  The token is `mcp__plugin_<pluginName>_<serverKey>__*` — here plugin `ac2`,
  server key `ac2-channel`. The `/ac2:setup` command writes this for you. (If
  the exact token ever differs, "always allow" a tool once and copy whatever
  Claude Code writes to `settings.local.json`.)

- **Or bypass permission prompts entirely** by launching with
  `--dangerously-skip-permissions` — simpler, but it skips prompts for *all*
  tools (Bash/Write/…), so only use it in a directory you trust.

## Configuration (env)

| Variable | Default | Purpose |
|----------|---------|---------|
| `AC2_LIQUID_AUTH_SERVER` | `https://liquidauth.goplausible.xyz` | Liquid Auth relay URL |
| `AC2_AGENT_DID` | auto-generated `did:key` | Stable DID for this agent (persisted at `~/.claude/channels/ac2/agent-key.json`) |
| `AC2_RTC_CONFIG` | — | JSON `RTCConfiguration` overrides (e.g. custom ICE servers) |

State lives under `~/.claude/channels/ac2/` (agent key, paired-peer record).

## Tools

| Tool | Purpose |
|------|---------|
| `reply` | Send a chat message back to the wallet user |
| `ac2_pair` | Start/refresh pairing — returns QR + deep link |
| `ac2_status` | Connection status (read-only) |
| `ac2_forget` | Unpair the wallet (destructive) |
| `ac2_version` | Plugin + SDK versions |
| `ac2_sign` | Ask the wallet to sign bytes |
| `ac2_capabilities` | Enumerate the wallet's identities/accounts |
| `ac2_verify_*` | Verify signatures (raw/message/typed-data/transaction across Ed25519/secp256k1, Algorand/EVM/Solana) |

## Publishing / non-dev install

To run without `--dangerously-load-development-channels`, the plugin must be on
the channel allowlist — either Anthropic's official `claude-plugins-official`
(partner-coordinated) or an org's `allowedChannelPlugins` managed setting
(Team/Enterprise). See the channels docs for details.
