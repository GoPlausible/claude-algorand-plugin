---
name: haystack-router-development
description: "Build applications with Haystack Router SDK (@txnlab/haystack-router) — a DEX aggregator and smart order routing protocol on Algorand. Use when developing swap UIs with React and use-wallet, writing Node.js automation scripts, integrating RouterClient for quotes and swaps, or working with the SDK API surface, middleware, fees, and referral program."
---

# Haystack Router (Development)

This skill is for **building applications** that integrate the `@txnlab/haystack-router` SDK — React swap UIs, Node.js automation scripts, custom signing flows, middleware, and direct RouterClient usage. No Algorand MCP tools are involved.

> **Want to execute swaps** through the Algorand MCP server instead of building an app? Use the **haystack-router-interaction** skill — it provides Algorand MCP tools for quoting, swapping, and opt-in checking with wallet-based signing.

Haystack Router is a DEX aggregator and smart order routing protocol on Algorand. It finds optimal swap routes across multiple DEXes (Tinyman V2, Pact, Folks) and LST protocols (tALGO, xALGO), then executes them atomically through on-chain smart contracts.

## Package

`@txnlab/haystack-router` — TypeScript SDK for DEX-aggregated swaps. Requires `algosdk` (v3+) as a peer dependency.

```bash
npm install @txnlab/haystack-router algosdk
```

**API key required.** A free tier key is available for immediate use — see [configuration.md](references/configuration.md) for details.

## SDK Core Flow

```typescript
import { RouterClient } from '@txnlab/haystack-router'

// 1. Initialize
const router = new RouterClient({
  apiKey: '1b72df7e-1131-4449-8ce1-29b79dd3f51e', // Free tier (60 requests/min)
})

// 2. Get a quote
const quote = await router.newQuote({
  fromASAID: 0, // ALGO
  toASAID: 31566704, // USDC
  amount: 1_000_000, // 1 ALGO (base units)
  address: activeAddress,
})

// 3. Execute the swap (use-wallet signer for browser, custom signer for Node.js)
const swap = await router.newSwap({
  quote,
  address: activeAddress,
  signer: transactionSigner,
  slippage: 1, // 1%
})
const result = await swap.execute()
```

The SDK is the **only** supported integration path. Do not call the API directly.

## CRITICAL: Swap Direction (`type` parameter)

The `type` parameter determines whether `amount` is the exact input or the exact output. **Getting this wrong means the user spends or receives the wrong amount.**

- **`fixed-input`** (default): `amount` = **exact input** the user spends. Output varies by market price.
  - Use when: "sell 10 ALGO", "swap 10 ALGO to USDC", "use 10 ALGO to buy USDC"
- **`fixed-output`**: `amount` = **exact output** the user receives. Input varies by market price.
  - Use when: "buy 10 ALGO", "get me 10 USDC", "I want exactly 5 ALGO"

**Rule: "Buy X of Y" = fixed-output (user wants exactly X). "Sell/swap/use X of Y" = fixed-input (user spends exactly X). If ambiguous, ask.**

```typescript
// "Sell 10 ALGO for USDC" — user spends exactly 10 ALGO
const quote = await router.newQuote({
  fromASAID: 0, toASAID: 31566704,
  amount: 10_000_000, type: 'fixed-input',
})
// quote.quote = USDC received (variable)

// "Buy 10 USDC with ALGO" — user receives exactly 10 USDC
const quote = await router.newQuote({
  fromASAID: 0, toASAID: 31566704,
  amount: 10_000_000, type: 'fixed-output',
})
// quote.amount = ALGO required (variable)
```

## Key Concepts

- **Amounts** are always in base units (microAlgos for ALGO, smallest unit for ASAs)
- **ASA IDs**: 0 = ALGO, 31566704 = USDC, etc.
- **Quote types**: `fixed-input` (default) — specify input amount; `fixed-output` — specify desired output
- **Slippage**: Percentage tolerance on output (e.g., 1 = 1%). Applied to the final output, not individual hops
- **Routing**: Supports multi-hop and parallel (combo) swaps for optimal pricing
- **Middleware**: Plugin system for custom pre/post-swap transactions (e.g., auto opt-out)

## Reference Files

Read the appropriate file based on the task:

| Task                                    | Reference                                               |
| --------------------------------------- | ------------------------------------------------------- |
| Install SDK, initialize RouterClient    | [getting-started.md](references/getting-started.md)     |
| Get swap quotes, display pricing        | [quotes.md](references/quotes.md)                       |
| Execute swaps with SDK signing          | [swaps.md](references/swaps.md)                         |
| Build React swap UI with use-wallet     | [react-integration.md](references/react-integration.md) |
| Automate swaps via Node.js scripts      | [node-automation.md](references/node-automation.md)     |
| RouterClient config, middleware, debug  | [configuration.md](references/configuration.md)         |
| Fee structure, referral program         | [fees-and-referrals.md](references/fees-and-referrals.md) |
| Full API surface and types              | [api-reference.md](references/api-reference.md)         |
| Migrate from @txnlab/deflex             | [migration.md](references/migration.md)                 |
