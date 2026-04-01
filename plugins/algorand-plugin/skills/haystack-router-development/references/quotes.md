# Quotes

## Getting a Quote

```typescript
import { RouterClient } from '@txnlab/haystack-router'

const router = new RouterClient({
  apiKey: '1b72df7e-1131-4449-8ce1-29b79dd3f51e', // Free tier (60 requests/min)
})

const quote = await router.newQuote({
  fromASAID: 0, // ALGO
  toASAID: 31566704, // USDC
  amount: 1_000_000, // 1 ALGO in base units
  address: activeAddress,
})
```

## Parameters

| Parameter           | Type               | Required | Description                                     |
| ------------------- | ------------------ | -------- | ----------------------------------------------- |
| `fromASAID`         | `number \| bigint` | Yes      | Input asset ID (0 = ALGO)                       |
| `toASAID`           | `number \| bigint` | Yes      | Output asset ID                                 |
| `amount`            | `number \| bigint` | Yes      | Amount in base units                            |
| `type`              | `string`           | No       | `'fixed-input'` (default) or `'fixed-output'`   |
| `address`           | `string`           | No       | User address (needed for auto opt-in detection) |
| `maxGroupSize`      | `number`           | No       | Max transactions in group (default: 16)         |
| `maxDepth`          | `number`           | No       | Max routing hops (default: 4)                   |
| `optIn`             | `boolean`          | No       | Include opt-in transaction for output asset     |
| `disabledProtocols` | `Protocol[]`       | No       | Protocols to exclude from routing               |

## Quote Response (SwapQuote)

`newQuote()` returns a `SwapQuote` (extends `FetchQuoteResponse`):

```typescript
quote.quote // bigint — expected output amount in base units
quote.amount // bigint — original input amount
quote.usdIn // number — USD value of input
quote.usdOut // number — USD value of output
quote.userPriceImpact // number | undefined — price impact %
quote.marketPriceImpact // number | undefined — market price impact %
quote.route // Route[] — routing path details
quote.flattenedRoute // Record<string, number> — protocol split percentages
quote.quotes // DexQuote[] — individual DEX quotes
quote.requiredAppOptIns // number[] — app IDs needing opt-in
quote.createdAt // number — timestamp (ms)
quote.address // string | undefined — user address if provided
```

## Quote Types — CRITICAL: Choosing the Right Direction

**Getting `type` wrong means the user spends or receives the wrong amount. Parse user intent carefully.**

| User says | `type` | `amount` is |
|-----------|--------|-------------|
| "Buy 10 ALGO with USDC" | `'fixed-output'` | 10_000_000 (the ALGO — desired output) |
| "Sell 10 ALGO for USDC" | `'fixed-input'` | 10_000_000 (the ALGO — exact spend) |
| "Swap 10 ALGO to USDC" | `'fixed-input'` | 10_000_000 (the ALGO — exact spend) |
| "Use 10 ALGO to buy USDC" | `'fixed-input'` | 10_000_000 (the ALGO — exact spend) |
| "Buy USDC for 10 ALGO" | `'fixed-input'` | 10_000_000 (the ALGO — exact spend) |

**Rule: "Buy X of Y" = fixed-output. "Sell/swap/use X of Y" = fixed-input. If ambiguous, ask the user.**

### Fixed-input (default): User specifies exact input, output varies

```typescript
// "Sell 10 ALGO for USDC" — user spends exactly 10 ALGO
const quote = await router.newQuote({
  fromASAID: 0,
  toASAID: 31566704,
  amount: 10_000_000, // Exact input: spend 10 ALGO
  type: 'fixed-input',
})
// quote.quote = USDC received (variable based on market)
```

### Fixed-output: User specifies exact output, input varies

```typescript
// "Buy 10 USDC with ALGO" — user receives exactly 10 USDC
const quote = await router.newQuote({
  fromASAID: 0,
  toASAID: 31566704,
  amount: 10_000_000, // Exact output: receive 10 USDC
  type: 'fixed-output',
})
// quote.amount = ALGO required to spend (variable based on market)
```

## Displaying Quote Data

```typescript
const fromDecimals = 6 // ALGO
const toDecimals = 6 // USDC

const outputHuman = Number(quote.quote) / 10 ** toDecimals
const inputHuman = Number(quote.amount) / 10 ** fromDecimals
const rate = outputHuman / inputHuman

console.log(`${inputHuman} ALGO → ${outputHuman} USDC`)
console.log(`Rate: 1 ALGO = ${rate.toFixed(4)} USDC`)
console.log(`USD in: $${quote.usdIn.toFixed(2)}`)
console.log(`USD out: $${quote.usdOut.toFixed(2)}`)

if (quote.userPriceImpact !== undefined) {
  console.log(`Price impact: ${quote.userPriceImpact.toFixed(2)}%`)
}
```

## Route Details

Each quote includes routing information showing how the swap is split:

```typescript
// Flattened view: protocol → percentage
for (const [protocol, pct] of Object.entries(quote.flattenedRoute)) {
  console.log(`${protocol}: ${pct}%`)
}
// e.g., "TinymanV2: 60%", "Pact: 40%"

// Detailed route with hops
for (const route of quote.route) {
  console.log(`${route.percentage}% of swap:`)
  for (const hop of route.path) {
    console.log(`  ${hop.in.unit_name} → ${hop.out.unit_name} via ${hop.name}`)
  }
}
```

## Asset Opt-In Detection

```typescript
// Option 1: Set autoOptIn on the client
const router = new RouterClient({
  apiKey: '1b72df7e-1131-4449-8ce1-29b79dd3f51e', // Free tier (60 requests/min)
  autoOptIn: true,
})
const quote = await router.newQuote({
  fromASAID: 0,
  toASAID: 31566704,
  amount: 1_000_000,
  address: activeAddress, // Required for auto opt-in
})

// Option 2: Check manually and pass optIn flag
const needsOptIn = await router.needsAssetOptIn(activeAddress, 31566704)
const quote = await router.newQuote({
  fromASAID: 0,
  toASAID: 31566704,
  amount: 1_000_000,
  optIn: needsOptIn,
})
```

## Lower-Level: fetchQuote()

`fetchQuote()` returns the raw `FetchQuoteResponse` without `SwapQuote` enhancements (no bigint coercion, no `createdAt`). Use `newQuote()` unless you need the raw response.

```typescript
const raw = await router.fetchQuote({
  fromASAID: 0,
  toASAID: 31566704,
  amount: 1_000_000,
})
// raw.quote is string | number (not bigint)
```
