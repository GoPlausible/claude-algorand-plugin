---
name: algorand-interaction
description: Interact with Algorand blockchain via the Algorand MCP server — wallet operations, ALGO/ASA transactions, smart contracts, account info, NFD lookups, atomic groups, Tinyman swaps, TEAL compilation, knowledge base. Use when user asks about Algorand wallet, balances, sending ALGO or tokens, asset opt-in, transactions, NFD names, DEX swaps, smart contracts, or account details.
---

# Algorand MCP Interaction

Interact with Algorand blockchain through the Algorand MCP server (101 tools across 11 categories).

## Key Characteristics

- **Secure signing** — use `wallet_*` tools to sign; private keys are never available to you
- **Multi-network** — supports `mainnet`, `testnet`, and `localnet`
- **Spending limits** — per-transaction (`allowance`) and daily (`dailyAllowance`) limits enforced by wallet

## Calling MCP Tools

MCP tools are **deferred** — you MUST use `ToolSearch` to load them before calling:

```
ToolSearch("+algorand wallet")                                    # Search by keyword — loads matching tools
ToolSearch("select:mcp__algorand-mcp__wallet_get_info")           # Load a specific tool by full name
```

Once loaded, call them normally:
```
mcp__algorand-mcp__wallet_get_info { "network": "testnet" }
mcp__algorand-mcp__make_payment_txn { "from": "ADDR", "to": "ADDR", "amount": 1000000, "network": "testnet" }
```

Full tool name pattern: `mcp__algorand-mcp__<tool_name>`. If you get "tool not found", use `ToolSearch("+algorand <keyword>")` to load it first.

## Session Start Checklist

**At EVERY session start:**

1. **Load tools first**: Call `ToolSearch("+algorand wallet")` to load wallet tools — MCP tools are deferred and MUST be loaded via ToolSearch before use
2. **Check wallet**: `mcp__algorand-mcp__wallet_get_info` with target `network` — verify an account exists and is active
3. **If no accounts**: Guide user to create one with `wallet_add_account` (load via ToolSearch first)
4. **If needs funding**: Generate ARC-26 QR with `generate_algorand_uri` or direct to testnet faucet: https://lora.algokit.io/testnet/fund
5. **If needs USDC funding**: Generate ARC-26 QR with `generate_algorand_uri` or direct to testnet faucet: https://faucet.circle.com/
6. **Confirm network**: Always confirm which network (`mainnet`, `testnet`, `localnet`) before transactions
7. **Load additional tools as needed**: Use `ToolSearch("+algorand <keyword>")` to load tools for the task at hand

## Network Selection

Every tool that touches the blockchain accepts a `network` parameter:

| Value | Description |
|-------|-------------|
| `mainnet` | Algorand mainnet (default) — **real value, exercise caution** |
| `testnet` | Algorand testnet — safe for development |
| `localnet` | Local dev network (requires `ALGORAND_LOCALNET_URL` env var) |

Default to `testnet` during development.

## Pre-Transaction Validation

Before ANY transaction:

1. **MBR Check**: Account needs 0.1 ALGO base + 0.1 per asset/app opt-in
2. **Asset Opt-In**: Verify with `api_algod_get_account_asset_info` before ASA transfers
3. **Fees**: Every txn costs 0.001 ALGO (1,000 microAlgos) minimum
4. **Balance Check**: Fetch current balance with `wallet_get_info` or `api_algod_get_account_info`
5. **Spending Limits**: Wallet enforces per-transaction (`allowance`) and daily (`dailyAllowance`) limits. Setting either to `0` means **unlimited**
6. **Order**: Fund account with ALGO first, then asset transactions

## Common Mainnet Assets

| Asset | ASA ID | Decimals |
|-------|--------|----------|
| ALGO | native | 6 |
| USDC | 31566704 | 6 |
| USDT | 312769 | 6 |
| goETH | 386192725 | 8 |
| goBTC | 386195940 | 8 |

> Always verify asset IDs on-chain — scam tokens use similar names.

## Amounts and Decimals

| Asset | Unit | 1 Whole Token = |
|-------|------|-----------------|
| ALGO | microAlgos | 1,000,000 |
| USDC (ASA 31566704) | micro-units | 1,000,000 (6 decimals) |
| Custom ASAs | base units | Depends on `decimals` field |

Always check asset's `decimals` field with `api_algod_get_asset_by_id` before computing amounts.

## Transaction Types

- **pay**: ALGO transfers → `make_payment_txn`
- **axfer**: Asset transfers, opt-in, clawback → `make_asset_transfer_txn`
- **acfg**: Asset create/configure/destroy → `make_asset_create_txn`, `make_asset_config_txn`, `make_asset_destroy_txn`
- **afrz**: Asset freeze/unfreeze → `make_asset_freeze_txn`
- **appl**: Smart contract calls → `make_app_create_txn`, `make_app_call_txn`, `make_app_update_txn`, `make_app_delete_txn`, `make_app_optin_txn`, `make_app_closeout_txn`, `make_app_clear_txn`
- **keyreg**: Consensus key registration → `make_keyreg_txn`

## Wallet Transaction Workflow (Recommended)

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `wallet_get_info` | Verify active account, check balance |
| 2 | Query tools | Get blockchain data (account info, asset info, etc.) |
| 3 | `make_*_txn` | Build the transaction |
| 4 | `wallet_sign_transaction` | Sign with active wallet account (enforces limits) |
| 5 | `send_raw_transaction` | Submit signed transaction to network |
| 6 | Query tools | Verify result on-chain |

### One-Step Asset Opt-In

For asset opt-ins, use the shortcut:
```
wallet_optin_asset { assetId: 31566704, network: "testnet" }
```

## External Key Transaction Workflow

When the user provides their own secret key (not using the wallet):

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `make_*_txn` | Build the transaction |
| 2 | `sign_transaction` | Sign with provided secret key hex |
| 3 | `send_raw_transaction` | Submit signed transaction |

## Atomic Group Transaction Workflow

For atomic (all-or-nothing) multi-transaction groups:

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `make_*_txn` (multiple) | Build each transaction |
| 2 | `assign_group_id` | Assign group ID to all transactions |
| 3 | `wallet_sign_transaction_group` | Sign all transactions in group with wallet |
| 4 | `send_raw_transaction` | Submit all signed transactions |

## Tool Categories

**Wallet** (10): `wallet_add_account`, `wallet_remove_account`, `wallet_list_accounts`, `wallet_switch_account`, `wallet_get_info`, `wallet_get_assets`, `wallet_sign_transaction`, `wallet_sign_transaction_group`, `wallet_sign_data`, `wallet_optin_asset`

**Account** (8): `create_account`, `rekey_account`, `mnemonic_to_mdk`, `mdk_to_mnemonic`, `secret_key_to_mnemonic`, `mnemonic_to_secret_key`, `seed_from_mnemonic`, `mnemonic_from_seed`

**Utility** (13): `ping`, `validate_address`, `encode_address`, `decode_address`, `get_application_address`, `bytes_to_bigint`, `bigint_to_bytes`, `encode_uint64`, `decode_uint64`, `verify_bytes`, `sign_bytes`, `encode_obj`, `decode_obj`

**Transaction** (18): `make_payment_txn`, `make_keyreg_txn`, `make_asset_create_txn`, `make_asset_config_txn`, `make_asset_destroy_txn`, `make_asset_freeze_txn`, `make_asset_transfer_txn`, `make_app_create_txn`, `make_app_update_txn`, `make_app_delete_txn`, `make_app_optin_txn`, `make_app_closeout_txn`, `make_app_clear_txn`, `make_app_call_txn`, `assign_group_id`, `sign_transaction`, `encode_unsigned_transaction`, `decode_signed_transaction`

**Algod** (5): `compile_teal`, `disassemble_teal`, `send_raw_transaction`, `simulate_raw_transactions`, `simulate_transactions`

**Algod API** (13): `api_algod_get_account_info`, `api_algod_get_account_application_info`, `api_algod_get_account_asset_info`, `api_algod_get_application_by_id`, `api_algod_get_application_box`, `api_algod_get_application_boxes`, `api_algod_get_asset_by_id`, `api_algod_get_pending_transaction`, `api_algod_get_pending_transactions_by_address`, `api_algod_get_pending_transactions`, `api_algod_get_transaction_params`, `api_algod_get_node_status`, `api_algod_get_node_status_after_block`

**Indexer API** (17): `api_indexer_lookup_account_by_id`, `api_indexer_lookup_account_assets`, `api_indexer_lookup_account_app_local_states`, `api_indexer_lookup_account_created_applications`, `api_indexer_search_for_accounts`, `api_indexer_lookup_applications`, `api_indexer_lookup_application_logs`, `api_indexer_search_for_applications`, `api_indexer_lookup_application_box`, `api_indexer_lookup_application_boxes`, `api_indexer_lookup_asset_by_id`, `api_indexer_lookup_asset_balances`, `api_indexer_lookup_asset_transactions`, `api_indexer_search_for_assets`, `api_indexer_lookup_transaction_by_id`, `api_indexer_lookup_account_transactions`, `api_indexer_search_for_transactions`

**NFDomains** (6): `api_nfd_get_nfd`, `api_nfd_get_nfds_for_addresses`, `api_nfd_get_nfd_activity`, `api_nfd_get_nfd_analytics`, `api_nfd_browse_nfds`, `api_nfd_search_nfds`

**Tinyman DEX** (9): `api_tinyman_get_pool`, `api_tinyman_get_pool_analytics`, `api_tinyman_get_pool_creation_quote`, `api_tinyman_get_liquidity_quote`, `api_tinyman_get_remove_liquidity_quote`, `api_tinyman_get_swap_quote`, `api_tinyman_get_asset_optin_quote`, `api_tinyman_get_validator_optin_quote`, `api_tinyman_get_validator_optout_quote`

**ARC-26 URI** (1): `generate_algorand_uri`

**Knowledge Base** (1): `get_knowledge_doc`

## Pagination

API responses are paginated. All API tools accept optional `itemsPerPage` (default 10) and `pageToken` parameters. Pass `pageToken` from a previous response to fetch the next page.

## x402 Payment Workflow

When an HTTP request returns 402 with `PaymentRequirements`, use the atomic group transaction pattern to build the payment:

1. Parse the 402 response body — extract `accepts[]` array, choose an entry matching a supported network/asset
2. Map CAIP-2 network identifier from `accepts[].network` to `testnet` or `mainnet`
3. Check wallet with `wallet_get_info`
4. Build fee payer transaction (0-amount from facilitator's `feePayer` address, `fee: 2000`, `flatFee: true` — covers fees for both txns)
5. Build payment transaction (ALGO or ASA transfer to `payTo`, `fee: 0`, `flatFee: true` — facilitator covers fee)
6. Group both transactions with `assign_group_id`
7. Sign only the payment transaction (index 1) with `wallet_sign_transaction` — leave fee payer unsigned
8. Encode the unsigned fee payer transaction (index 0) with `encode_unsigned_transaction`
9. Construct PAYMENT-SIGNATURE JSON payload — **must include `accepted` field** (the exact `accepts[]` entry chosen)
10. Base64-encode the JSON and retry with `curl -H "PAYMENT-SIGNATURE: <base64>"`

**Critical**: The `accepted` field is REQUIRED. It must be an exact copy of the `accepts[]` entry you chose to pay with (including all fields: scheme, network, price, payTo, asset, maxAmountRequired, extra, etc.). Without it, the server cannot match your payment and will reject with 402.

**Critical — Base64 blob handling**: When constructing the PAYMENT-SIGNATURE via shell/curl, NEVER manually copy-paste base64 blob strings inline into a JSON string. Base64 blobs are long and even a single character corruption (e.g., `5` → `4`) causes "signature does not match sender" errors. ALWAYS store each blob (`bytes` from `encode_unsigned_transaction`, `blob` from `wallet_sign_transaction`) in a separate shell variable first, then interpolate via `${VAR}`. Use `printf '%s'` (not `echo`) and `tr -d '\n'` to strip macOS base64 newlines.

For the full step-by-step workflow, load the `algorand-x402-payment` skill or see [references/examples-algorand-mcp.md](references/examples-algorand-mcp.md).

## References

For detailed tool documentation:
- **Tool Reference**: See [references/algorand-mcp.md](references/algorand-mcp.md)

For workflow examples (including x402 payment):
- **Examples**: See [references/examples-algorand-mcp.md](references/examples-algorand-mcp.md)

## NFD Important Note

When using NFD (`.algo` names), always use the `depositAccount` field from the NFD response for transactions, NOT other address fields.

## Security

- **Mainnet = real value** — always confirm with user before mainnet transactions
- Never log, display, or store mnemonics or secret keys — use `wallet_*` tools for signing
- Verify recipient addresses with `validate_address` — transactions are irreversible
- Verify asset IDs on-chain — scam tokens use similar names
- Respect wallet spending limits — if rejected, inform user rather than bypassing

## Links

- GoPlausible: https://goplausible.com
- Algorand: https://algorand.co
- Algorand x402: https://x402.goplausible.xyz
- Algorand x402 test endpoints: https://example.x402.goplausible.xyz/
- Algorand x402 Facilitator: https://facilitator.goplausible.xyz
- Testnet Faucet: https://lora.algokit.io/testnet/fund
- Testnet USDC Faucet: https://faucet.circle.com/
- Algorand Developer Docs: https://dev.algorand.co/
- Algorand Developer Docs Github : https://github.com/algorandfoundation/devportal
- Algorand Developer Examples Github : https://github.com/algorandfoundation/devportal-code-examples
- [GoPlausible x402-avm Documentation and Example code](https://github.com/GoPlausible/.github/blob/main/profile/algorand-x402-documentation/README.md)
- [GoPlausible x402-avm Examples template Projects](https://github.com/GoPlausible/x402-avm/tree/branch-v2-algorand-publish/examples/)
- [CAIP-2 Specification](https://github.com/ChainAgnostic/CAIPs/blob/main/CAIPs/caip-2.md)
- [Coinbase x402 Protocol](https://github.com/coinbase/x402)
