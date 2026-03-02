# Algorand Plugin for Claude Code

Algorand blockchain integration for [Claude Code](https://claude.ai/code) — by [GoPlausible](https://goplausible.com)

## Features

- **MCP Server**: Bundled `@goplausible/algorand-mcp` (101 tools) — wallet, transactions, smart contracts, TEAL, indexer, DEX, NFD, knowledge base
- **x402 Payment Protocol**: Claude autonomously pays for x402-protected HTTP resources using Algorand
- **7 Skills**:
  - `algorand-development` — AlgoKit CLI, project creation, general workflows
  - `algorand-typescript` — TypeScript smart contracts (Algorand TypeScript / PuyaTs)
  - `algorand-python` — Python smart contracts (Algorand Python / PuyaPy)
  - `algorand-interaction` — Blockchain interaction via MCP (wallet, transactions, swaps, NFD)
  - `algorand-x402-typescript` — Build x402 payment servers/clients in TypeScript
  - `algorand-x402-python` — Build x402 payment servers/clients in Python
  - `algorand-x402-payment` — Runtime x402 payments (Claude as autonomous client)
- **Agent**: `algorand-agent` for complex multi-step Algorand tasks
- **Multi-network**: `mainnet`, `testnet`, `localnet`
- **Secure wallet**: Per-transaction and daily spending limits, private keys never exposed

## Installation

### From marketplace (recommended)

This repo is both the plugin **and** a self-contained marketplace. Add it, then install:

**From GitHub (for any user):**

```
/plugin marketplace add GoPlausible/claude-algorand-plugin
/plugin install algorand-plugin@goplausible-algorand
```

**From local clone (for development):**

```bash
git clone https://github.com/GoPlausible/claude-algorand-plugin.git
```
Then inside Claude Code:
```
/plugin marketplace add /path/to/claude-algorand-plugin
/plugin install algorand-plugin@goplausible-algorand
```

### From directory (development/testing)

For quick testing without installation, load directly:

```bash
claude --plugin-dir /path/to/claude-algorand-plugin
```

> **Note**: `--plugin-dir` is session-scoped — the plugin is only loaded for that session. Use marketplace installation for persistent access from any directory.

### Prerequisites

The plugin's MCP server runs via `npx`, so you need Node.js installed:

```bash
npm install -g @goplausible/algorand-mcp
# or let npx handle it automatically on first use
```

### Validate installation

After installing, verify the plugin is loaded:

```
/plugin list                  # Should show algorand-plugin
/mcp                          # Should show algorand-mcp server
/algorand-plugin:algorand-interaction   # Should load the interaction skill
```

## Auto-Allow MCP Tools (Recommended)

By default, Claude will prompt for permission on every MCP tool call. To auto-allow all Algorand MCP tools, add this to your settings:

### Project-level (recommended)

Add to `.claude/settings.json` in your project root:

```json
{
  "permissions": {
    "allow": [
       "mcp__algorand-mcp__*",
      "claude mcp:*",
    ]
  }
}
```

### User-level (all projects)

Add to `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
       "mcp__algorand-mcp__*",
      "claude mcp:*",
    ]
  }
}
```

The pattern `mcp__algorand-mcp__*` matches all 101 tools from the Algorand MCP server.

> **Note**: Plugins cannot ship their own permission allowlists — this is by design. Permissions are a trust decision made by the user, not the plugin author.

## Tool Loading (Important)

This plugin provides 101 MCP tools. Claude Code automatically **defers** large tool sets to save context window space. This means Claude must use `ToolSearch` to load tools before calling them — the plugin handles this via SessionStart hooks and skill instructions.

If you prefer all 101 tools loaded directly into context (no ToolSearch needed), set this environment variable:

```bash
ENABLE_TOOL_SEARCH=false claude --plugin-dir /path/to/claude-algorand-plugin
```

Or add to `~/.claude/settings.json`:

```json
{
  "env": {
    "ENABLE_TOOL_SEARCH": "false"
  }
}
```

> **Trade-off**: Disabling tool search loads all 101 tool definitions into context at session start, using more context window space but ensuring tools are always immediately available.

## MCP Tool Categories (101 tools)

| Category | Count | Examples |
|----------|-------|---------|
| Wallet | 10 | `wallet_get_info`, `wallet_sign_transaction`, `wallet_optin_asset` |
| Account Management | 8 | `create_account`, `rekey_account`, `mnemonic_to_secret_key` |
| Utility | 13 | `ping`, `validate_address`, `sign_bytes`, `encode_obj` |
| Transaction | 18 | `make_payment_txn`, `make_asset_transfer_txn`, `assign_group_id` |
| Algod | 5 | `compile_teal`, `send_raw_transaction`, `simulate_transactions` |
| Algod API | 13 | `api_algod_get_account_info`, `api_algod_get_transaction_params` |
| Indexer API | 17 | `api_indexer_search_for_transactions`, `api_indexer_lookup_asset_by_id` |
| NFDomains | 6 | `api_nfd_get_nfd`, `api_nfd_search_nfds` |
| Tinyman AMM | 9 | `api_tinyman_get_swap_quote`, `api_tinyman_get_pool` |
| ARC-26 URI | 1 | `generate_algorand_uri` |
| Knowledge | 1 | `get_knowledge_doc` |

## x402 Payment Protocol

When Claude encounters an HTTP 402 response from an x402-protected resource, it can autonomously:

1. Parse the `PaymentRequirements` from the 402 response
2. Build an atomic transaction group (fee payer + payment)
3. Sign the payment via the secure wallet
4. Retry the request with the `PAYMENT-SIGNATURE` header
5. Return the protected resource content

Test endpoint: `https://example.x402.goplausible.xyz/`

## Skills Overview

| Skill | Purpose |
|-------|---------|
| `algorand-development` | AlgoKit CLI, project creation, general workflows |
| `algorand-typescript` | TypeScript smart contracts (Algorand TypeScript / PuyaTs) |
| `algorand-python` | Python smart contracts (Algorand Python / PuyaPy) |
| `algorand-interaction` | MCP-based blockchain interaction (wallet, txns, DEX, NFD) |
| `algorand-x402-typescript` | x402 payments in TypeScript |
| `algorand-x402-python` | x402 payments in Python |
| `algorand-x402-payment` | Runtime x402 payments (Claude as autonomous client) |

## Links

- **GoPlausible**: https://goplausible.com
- **Algorand**: https://algorand.co
- **Algorand Developer Docs**: https://dev.algorand.co/
- **Algorand x402**: https://x402.goplausible.xyz
- **x402 Test Endpoints**: https://example.x402.goplausible.xyz/
- **x402 Facilitator**: https://facilitator.goplausible.xyz
- **Testnet Faucet**: https://lora.algokit.io/testnet/fund

## License

MIT © [GoPlausible](https://goplausible.com)
