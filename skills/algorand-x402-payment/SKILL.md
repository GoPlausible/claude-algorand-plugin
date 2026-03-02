---
name: algorand-x402-payment
description: Execute x402 payments at runtime using Algorand MCP tools. Use when Claude encounters HTTP 402 Payment Required responses, needs to access paid APIs, or the user asks to fetch x402-protected resources.
---

# x402 Runtime Payment

You (Claude) are the x402 client. When you encounter an HTTP 402 response with `PaymentRequirements`, use MCP tools to build, sign, and submit an Algorand payment, then retry the request.

## When to Use

- HTTP response with status `402` and body containing `x402Version` and `accepts[]`
- User asks to access a paid/protected API or resource
- User mentions x402, payment-required, or paid endpoint access

## Quick Flow

1. `curl` the URL → detect 402, parse `accepts[]`
2. Choose an `accepts` entry → extract `network`, `payTo`, `amount`, `asset`, `feePayer`
3. Map CAIP-2 network → MCP network parameter
4. `wallet_get_info` → verify wallet, get address
5. Check asset opt-in (ASA only) → `wallet_optin_asset` if needed
6. `make_payment_txn` → fee payer (from=feePayer, to=feePayer, amount=0, fee=2000, flatFee=true)
7. `make_payment_txn` or `make_asset_transfer_txn` → payment (fee=0, flatFee=true)
8. `assign_group_id` → group [feePayer@0, payment@1]
9. `wallet_sign_transaction` → sign payment only (index 1)
10. `encode_unsigned_transaction` → encode fee payer (index 0)
11. Construct PAYMENT-SIGNATURE JSON (include `accepted` field)
12. `curl -H 'PAYMENT-SIGNATURE: <base64>'` → retry, get 200

## CAIP-2 Network Mapping

| CAIP-2 Genesis Hash | MCP Network |
|----------------------|-------------|
| `SGO1GKSzyE7IEPItTxCByw9x8FmnrCDexi9/cOUJOiI=` | `"testnet"` |
| `wGHE2Pwdvd7S12BL5FaOP20EGYesN73ktiC1qzkkit8=` | `"mainnet"` |

## Asset IDs

| Asset | Testnet | Mainnet | Decimals |
|-------|---------|---------|----------|
| ALGO | `0` (native) | `0` (native) | 6 |
| USDC | `10458941` | `31566704` | 6 |

## Critical Rules

1. **`accepted` field is REQUIRED** — Include a verbatim copy of the chosen `accepts[]` entry in the PAYMENT-SIGNATURE JSON. Without it, the server rejects.
2. **Group order**: feePayer at index 0, payment at index 1. `paymentIndex: 1`.
3. **Only sign the payment** (index 1) — the facilitator signs the fee payer server-side.
4. **`flatFee: true`** on both transactions — prevents SDK from overriding fees.
5. **Mainnet = real money** — always confirm with the user before mainnet payments.
6. **One retry only** — if the retry also returns 402, stop and report the error.
7. **`feePayer` address** comes from `extra.feePayer` in the PaymentRequirements.

## Test Endpoint

`https://example.x402.goplausible.xyz/` — testnet x402-protected resources for testing.

## References

- [x402-payment-flow.md](references/x402-payment-flow.md) — Complete step-by-step MCP tool recipe with worked example
- [x402-payment-reference.md](references/x402-payment-reference.md) — Protocol spec, header format, schemas, CAIP-2 identifiers
