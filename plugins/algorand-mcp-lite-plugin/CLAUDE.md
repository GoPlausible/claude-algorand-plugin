# Algorand Remote MCP Lite Plugin

This plugin provides **Algorand Remote MCP Lite (Wallet Edition)** — a managed remote MCP server with 59 tools for wallet operations, ALGO/ASA transactions, atomic groups, blockchain queries, NFD lookups, Haystack Router DEX-aggregated swaps, Alpha Arcade prediction markets, Pera asset verification, ARC-26 QR codes, transaction receipts, and rich UI widgets.

## Required — Load Skill First

**Before using ANY Algorand Remote MCP Lite tool, ALWAYS load and read the `algorand-mcp-lite` skill first.** This skill contains all tool references, workflows, security rules, and best practices needed to use the MCP server correctly.

```
/algorand-mcp-lite-plugin:algorand-mcp-lite
```

Load and read this skill at the start of every session where you interact with Algorand Remote MCP Lite tools. Do NOT attempt to use the tools without loading the skill first — it defines critical transaction validation, swap direction rules, error handling, and response handling behaviors.

## MCP Server

The plugin connects to the Algorand Remote MCP Lite server at `https://algorandmcplite.goplausible.xyz/sse` via `mcp-remote`. All 59 tools are available through this connection. Server-side signing is handled via HashiCorp Vault — no private keys needed.

## Key Rules

- Always start with `wallet_get_info` before any blockchain operation.
- Mainnet uses real assets with real value — exercise caution but do not ask user for confirmation on every transaction — the skill handles this with built-in rules.
- Check the skill for tool references, workflows, and swap direction rules before proceeding.
- After any transaction, always present the txId as a clickable explorer link (mainnet: `https://allo.info/tx/{txId}`, testnet: `https://lora.algokit.io/testnet/transaction/{txId}`).
