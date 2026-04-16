---
name: algorand-interaction
description: Interact with Algorand blockchain via the Algorand MCP server — wallet operations, ALGO/ASA transactions, smart contracts, account info, NFD lookups, atomic groups, Tinyman swaps, Haystack Router best-price swaps, Alpha Arcade prediction markets, Pera asset verification, TEAL compilation, knowledge base. Use when user asks about Algorand wallet, balances, sending ALGO or tokens, asset opt-in, transactions, NFD names, DEX swaps, DEX aggregation, best-price routing, prediction markets, Alpha Arcade, asset verification, smart contracts, or account details.
---

# Algorand MCP Interaction

Interact with Algorand blockchain through the Algorand MCP server (122 tools across 14 categories).

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
4. **If needs funding**: Generate ARC-26 QR with `generate_algorand_qrcode` or direct to testnet faucet: https://lora.algokit.io/testnet/fund
5. **If needs USDC funding**: Generate ARC-26 QR with `generate_algorand_qrcode` or direct to testnet faucet: https://faucet.circle.com/
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

> Always verify asset IDs on-chain — scam tokens use similar names. Use `api_pera_asset_verification_status` to check verification tier before transacting unknown assets.

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
| 7 | — | **Present txId as explorer link** (mainnet: `https://allo.info/tx/{txId}`, testnet: `https://lora.algokit.io/testnet/transaction/{txId}`) |

> **Post-transaction display**: After `send_raw_transaction` (or any tool that returns a txId), always present the transaction ID to the user as a clickable explorer link. Mainnet: `https://allo.info/tx/{txId}`. Testnet: `https://lora.algokit.io/testnet/transaction/{txId}`. For multiple txIds (atomic groups, swaps), show each as a separate link.

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
| 4 | — | **Present txId as explorer link** (see Post-transaction display note above) |

## Atomic Group Transaction Workflow

For atomic (all-or-nothing) multi-transaction groups:

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `make_*_txn` (multiple) | Build each transaction |
| 2 | `assign_group_id` | Assign group ID to all transactions |
| 3 | `wallet_sign_transaction_group` | Sign all transactions in group with wallet |
| 4 | `send_raw_transaction` | Submit all signed transactions |
| 5 | — | **Present each txId as explorer link** (see Post-transaction display note above) |

## Best-Price Swap via Haystack Router (DEX Aggregator)

Haystack Router aggregates quotes across multiple Algorand DEXes (Tinyman, Pact, Folks) and LST protocols (tALGO, xALGO) to find the optimal swap route.

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `wallet_get_info` | Verify active account, check balance |
| 2 | `api_haystack_needs_optin` | Check if address needs opt-in for the target asset |
| 3 | `wallet_optin_asset` | Opt-in if needed |
| 4 | `api_haystack_get_swap_quote` | Preview best-price quote — show user output, USD values, route, price impact |
| 5 | User confirms | Always confirm before executing |
| 6 | `api_haystack_execute_swap` | All-in-one: quote + sign via wallet + submit + confirm |
| 7 | — | **Present each txId from response as explorer link** (mainnet: `https://allo.info/tx/{txId}`, testnet: `https://lora.algokit.io/testnet/transaction/{txId}`) |

> **CRITICAL — Swap direction (`type` parameter):**
> - **"Buy 10 ALGO"** → user wants exactly 10 ALGO **out** → `type: "fixed-output"`, `amount` = 10000000
> - **"Sell/swap 10 ALGO"** → user spends exactly 10 ALGO → `type: "fixed-input"`, `amount` = 10000000
> - **"Buy USDC for 10 ALGO"** → user spends exactly 10 ALGO → `type: "fixed-input"`, `amount` = 10000000
> - Rule: **"buy X of Y" = fixed-output**. **"sell/swap/use X of Y" = fixed-input**. If ambiguous, ask.

> For detailed Haystack Router workflows (batch swaps, configuration, slippage guidance), load the `haystack-router-interaction` skill.
> For building swap UIs or integrating the `@txnlab/haystack-router` SDK, load the `haystack-router-development` skill.

## Alpha Arcade Prediction Markets

Alpha Arcade provides on-chain prediction markets on Algorand. All collateral and payouts are in USDC (ASA 31566704). Prices and quantities use **microunits** (1,000,000 = $1.00 or 1 share).

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `wallet_get_info` | Verify active account, check ALGO + USDC balances |
| 2 | `alpha_get_live_markets` | Browse available markets (or `alpha_get_reward_markets` for rewarded markets) |
| 3 | `alpha_get_orderbook` | Check available liquidity for a specific market |
| 4 | `alpha_create_market_order` | Auto-match at best price (fills immediately → becomes position) |
| 4alt | `alpha_create_limit_order` | Place at specific price (rests on orderbook until matched) |
| 5 | `alpha_get_positions` | Check token balances (filled orders) |
| 5alt | `alpha_get_open_orders` | Check unfilled limit orders |

> **Post-trade display**: After `alpha_create_market_order` or `alpha_create_limit_order`, present any returned txIds as clickable explorer links. Mainnet: `https://allo.info/tx/{txId}`. Testnet: `https://lora.algokit.io/testnet/transaction/{txId}`.

> **CRITICAL — Prices are microunits, NOT percentages**: `yesProb`/`noProb` range 0–1,000,000. $0.50 = 500,000. Dividing by 100 instead of 1,000,000 causes uint64 overflow and transaction failures.

> **Orders require both ALGO and USDC**: ~0.957 ALGO locked per order (MBR) + USDC collateral for buys. "Overspend" errors mean insufficient ALGO or USDC.

> **Market orders become positions, not open orders**: After fill, check `alpha_get_positions` — not `alpha_get_open_orders`.

> **Multi-choice markets**: Trade using `options[].marketAppId`, not the parent market ID.

> For detailed Alpha Arcade workflows (orderbook mechanics, collateral, amend/cancel, split/merge, claiming), load the `alpha-arcade-interaction` skill.

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

**Haystack Router** (3): `api_haystack_get_swap_quote`, `api_haystack_execute_swap`, `api_haystack_needs_optin`

**Pera Asset Verification** (3): `api_pera_asset_verification_status`, `api_pera_verified_asset_details`, `api_pera_verified_asset_search`

**Alpha Arcade** (15): Read: `alpha_get_live_markets`, `alpha_get_reward_markets`, `alpha_get_market`, `alpha_get_orderbook`, `alpha_get_open_orders`, `alpha_get_positions`. Trade: `alpha_create_limit_order`, `alpha_create_market_order`, `alpha_cancel_order`, `alpha_amend_order`, `alpha_propose_match`, `alpha_split_shares`, `alpha_merge_shares`, `alpha_claim`

> For detailed Alpha Arcade workflows (orderbook mechanics, collateral, limit vs market orders), load the `alpha-arcade-interaction` skill.

**ARC-26 URI** (1): `generate_algorand_qrcode`

## QR Code Display (ARC-26 URI)

`generate_algorand_qrcode` generates an Algorand payment URI and QR code per ARC-26 specification via QRClaw service.

**Parameters:**
| Parameter | Required | Description |
|-----------|----------|-------------|
| `address` | Yes | Receiver Algorand address |
| `label` | No | Payment label |
| `amount` | No | Amount in microunits (e.g. 1000000 = 1 ALGO or 1 USDC) |
| `asset` | No | ASA ID for asset transfers; omit or 0 for ALGO |
| `note` | No | Payment note |
| `xnote` | No | Exclusive immutable note |

**Returns:**
- `qr` — UTF-8 text QR code (terminal-friendly)
- `uri` — the `algorand://` URI string
- `link` — shareable hosted QR URL (via QRClaw service)
- `expires_in` — link validity period

After calling the tool, **extract and paste the QR code directly in your response**.
**Always include all of these in your reply:**

1. **UTF-8 QR block** — Unicode block characters from `qr`. Paste inside a code block.
2. **URI string** — always show this, users need it for wallet deep links.
3. **Shareable link** — the hosted QR URL from `link`, so users can share or open it in a browser.



**Knowledge Base** (1): `get_knowledge_doc`

## Pagination

API responses are paginated. All API tools accept optional `itemsPerPage` (default 10) and `pageToken` parameters. Pass `pageToken` from a previous response to fetch the next page.

## x402 Payment Workflow

**IMPORTANT: Do NOT attempt x402 payments using only this skill.** When you encounter an HTTP 402 response or need to pay for an x402-protected resource, you MUST load the `algorand-x402-payment` skill which contains the complete, authoritative payment flow.

This skill provides the MCP tools needed for x402 (wallet, transactions, signing), but the `algorand-x402-payment` skill has the correct payload format, critical rules, and step-by-step recipe.

For reference examples, see [references/examples-algorand-mcp.md](references/examples-algorand-mcp.md).

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
- [Haystack Router (TxnLab DEX Aggregator)](https://github.com/TxnLab/haystack-router)
- Alpha Arcade: https://alphaarcade.com
- Alpha Arcade API: https://platform.alphaarcade.com
