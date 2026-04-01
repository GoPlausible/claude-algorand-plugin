# Algorand Remote MCP Plugin

This plugin provides **Algorand Remote MCP ** — a managed remote MCP server with 95 tools and 14 resources for wallet operations, ALGO/ASA transactions, atomic groups, blockchain queries, NFD lookups, Haystack Router DEX-aggregated swaps, Alpha Arcade prediction markets, Pera asset verification, ARC-26 QR codes, transaction receipts, x402 transactions, AP2 mandates, Algorand development tools and much more.

## Required — Load Skill First

**Before using ANY Algorand Remote MCP tool, ALWAYS load and read the `algorand-remote-mcp` skill first.** This skill contains all tool references, workflows, security rules, and best practices needed to use the MCP server correctly.

```
/algorand-remote-mcp-plugin:algorand-remote-mcp
```

Load and read this skill at the start of every session where you interact with Algorand Remote MCP tools. Do NOT attempt to use the tools without loading the skill first — it defines critical transaction validation, swap direction rules, error handling, and response handling behaviors.

## MCP Server

The plugin connects to the Algorand Remote MCP server at `https://algorandmcp.goplausible.xyz/sse` via `mcp-remote`. All 95 tools and 14 resources are available through this connection. Server-side signing is handled via HashiCorp Vault — no private keys needed.

## Key Rules

- Always start with `wallet_get_info` before any blockchain operation.
- Mainnet uses real assets with real value — exercise caution but do not ask user for confirmation on every transaction — the skill handles this with built-in rules.
- Check the skill for tool references, workflows, and swap direction rules before proceeding.
