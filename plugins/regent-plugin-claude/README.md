# @goplausible/regent-plugin-claude

Regent channel plugin for **Claude Code**. Pairs Claude Code with a Regent wallet
(e.g. [Regent](https://liquidauth.goplausible.xyz/download/regent.apk)) over
Liquid Auth (FIDO2) + WebRTC DataChannels, giving you:

- **Two-way chat** — message Claude Code from your wallet; replies come back.
- **Cryptographic signing** — `regent_sign` asks the wallet to sign bytes (the user
  approves in-wallet with passkey/biometric); `regent_verify_*` verify signatures.
- **Capability discovery** — `regent_capabilities` enumerates the wallet's
  identities and per-chain accounts.
- **Remote permission approval (HITL)** — Claude Code tool-approval prompts
  (Bash/Write/Edit) are relayed into your wallet chat; reply `yes <id>` / `no <id>`
  to approve or deny from your phone.

This is a **Claude Code channel** — an MCP server (stdio) that pushes events into
a running Claude Code session, implementing the
[Claude Code channel contract](https://code.claude.com/docs/en/channels-reference)
over a Regent connection (FIDO2 pairing, WebRTC transport, signing, capability
discovery, and auto-reconnect).

## Requirements

- **Claude Code v2.1.80+** (v2.1.81+ for permission relay) on a claude.ai
  subscription (channels don't work with API-key auth).
- **Node 20+** (the runtime — `@roamhq/wrtc` is a native addon that loads in
  Node, not Bun/Deno).

## Installation

```bash
// Step 1: Add the marketplace (run in Claude Code)
/plugin marketplace add GoPlausible/claude-algorand-plugin

// Step 2: Install the plugin (run in Claude Code)
/plugin install regent-plugin-claude@goplausible-claude-plugins
```

## Setup

```bash
// Step 3: One-time setup — run /regent:setup in Claude Code.
// This automatically allows the plugin's Regent tools so you
// aren't prompted for permission on every message.
/regent:setup
```

`/regent:setup` applies the settings below for you. This is only the manual
alternative — add it to `~/.claude/settings.json` (or `.claude/settings.json`)
yourself if you'd rather not run the setup wizard. Note `channelsEnabled` and
`allowedChannelPlugins` are **top-level** settings keys (not nested under
`permissions`):

```json
{
  "permissions": {
    "allow": [
      "mcp__plugin_regent_regent-channel__*",
      "Read(/skills/**)",
      "Read(/references/**)"
    ]
  },
  "channelsEnabled": true,
  "allowedChannelPlugins": [
    {
      "marketplace": "goplausible-claude-plugins",
      "plugin": "regent"
    }
  ]
}
```

```bash
// Step 4: Start Claude Code with channels enabled (run in your terminal).
// This flag is REQUIRED — without it the Regent channel won't load and
// no messages can be passed between Claude and your wallet.
claude --dangerously-load-development-channels server:plugin:regent:regent-channel
```

```bash
// Step 5: Pair your Regent wallet (e.g. Regent) — run in Claude Code.
// Shows a QR code + deep link to scan with your wallet.
/regent:pair
```

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
#    the /regent:* slash commands AND the MCP server (from this package's .mcp.json,
#    via ${CLAUDE_PLUGIN_ROOT} — no hand-written config needed).
claude --plugin-dir /abs/path/to/packages/regent-plugin-claude \
       --dangerously-load-development-channels server:plugin:regent:regent-channel
```

The dev-channel flag takes the MCP **server id**, which for a plugin-provided
server is `plugin:<pluginName>:<serverKey>` → here `plugin:regent:regent-channel`
(plugin name `regent`, server key `regent-channel` from `.mcp.json`). If the id is wrong,
the server still connects and tools work, but the logs show
`Channel notifications skipped: server <id> not in --channels list` and inbound
chat is dropped. Confirm the exact id with `/mcp` or the stderr logs.

Then in the session: run `/regent:pair` (or ask "pair my wallet") — it shows a QR +
deep link, scan it with Regent, and you're connected. Slash commands:
`/regent:pair`, `/regent:status`, `/regent:capabilities`, `/regent:sign`, `/regent:version`,
`/regent:forget`, `/regent:help`.

## Permissions (avoid a prompt on every message)

Claude calls this plugin's tools (`reply`, `regent_sign`, `regent_verify_*`,
`regent_capabilities`, …) constantly, so you'll get a permission prompt each time
unless you pre-approve them. Do **one** of:

- **Allow-list the plugin's tools** (recommended) — add this to your
  `~/.claude/settings.json` (or project `.claude/settings.json`) under
  `permissions.allow`:

  ```json
  { "permissions": { "allow": ["mcp__plugin_regent_regent-channel__*"] } }
  ```

  The token is `mcp__plugin_<pluginName>_<serverKey>__*` — here plugin `regent`,
  server key `regent-channel`. The `/regent:setup` command writes this for you. (If
  the exact token ever differs, "always allow" a tool once and copy whatever
  Claude Code writes to `settings.local.json`.)

- **Or bypass permission prompts entirely** by launching with
  `--dangerously-skip-permissions` — simpler, but it skips prompts for *all*
  tools (Bash/Write/…), so only use it in a directory you trust.

## Configuration (env)

| Variable | Default | Purpose |
|----------|---------|---------|
| `REGENT_LIQUID_AUTH_SERVER` | `https://liquidauth.goplausible.xyz` | Liquid Auth relay URL |
| `REGENT_AGENT_DID` | auto-generated `did:key` | Stable DID for this agent (persisted at `~/.claude/channels/regent/agent-key.json`) |
| `REGENT_RTC_CONFIG` | — | JSON `RTCConfiguration` overrides (e.g. custom ICE servers) |

State lives under `~/.claude/channels/regent/` (agent key, paired-peer record).

## Tools

| Tool | Purpose |
|------|---------|
| `reply` | Send a chat message back to the wallet user |
| `regent_pair` | Start/refresh pairing — returns QR + deep link |
| `regent_status` | Connection status (read-only) |
| `regent_forget` | Unpair the wallet (destructive) |
| `regent_version` | Plugin + SDK versions |
| `regent_sign` | Ask the wallet to sign bytes |
| `regent_capabilities` | Enumerate the wallet's identities/accounts |
| `regent_verify_*` | Verify signatures (raw/message/typed-data/transaction across Ed25519/secp256k1, Algorand/EVM/Solana) |

## Publishing / non-dev install

To run without `--dangerously-load-development-channels`, the plugin must be on
the channel allowlist — either Anthropic's official `claude-plugins-official`
(partner-coordinated) or an org's `allowedChannelPlugins` managed setting
(Team/Enterprise). See the channels docs for details.
