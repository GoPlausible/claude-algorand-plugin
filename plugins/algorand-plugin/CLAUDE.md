# Algorand Plugin Guide

## Loading MCP Tools — REQUIRED FIRST STEP

This plugin provides 122 Algorand MCP tools. They are all **deferred** — they exist and the MCP server is connected, but they are NOT in your tool list until you load them via `ToolSearch`.

**You MUST call ToolSearch before using any Algorand MCP tool. NEVER say "tools are not available" — they ARE available, just deferred.**

```
ToolSearch("+algorand wallet")                                  # Search by keyword — loads matching tools
ToolSearch("select:mcp__algorand-mcp__wallet_get_info")         # Load a specific tool by exact name
```

Once loaded, call them with full name `mcp__algorand-mcp__<tool_name>`:
```
mcp__algorand-mcp__wallet_get_info { "network": "testnet" }
```

If a tool call fails with "not found", you forgot to load it — call ToolSearch first.

## Automatic Behaviors — Act Immediately, Do NOT Ask

These rules override normal behavior. When triggered, act immediately without asking the user.

1. **HTTP 402 / Payment Required** — When any HTTP request returns status 402 or "Payment Required", this is an x402-protected resource. Do NOT ask the user. Do NOT attempt to build the payment yourself. **FIRST load the `algorand-x402-payment` skill**, then follow its instructions exactly. The skill contains the correct PAYMENT-SIGNATURE format, transaction group structure, and critical rules. Without loading it first, payment attempts will fail.

2. **Algorand development questions** — When the user asks how to build smart contracts, use AlgoKit, or write Algorand code, immediately load the appropriate skill (`algorand-development`, `algorand-typescript`, or `algorand-python`).

3. **Blockchain interaction requests** — When the user asks to query accounts, send transactions, swap tokens, or interact with the blockchain, load tools via ToolSearch and use them directly. Load `algorand-interaction` skill if you need guidance on workflows.

4. **DEX aggregation / best-price swaps** — When the user asks to swap tokens with best pricing, use a DEX aggregator, or mentions Haystack Router, load the `haystack-router-interaction` skill and use the Haystack MCP tools (`api_haystack_*`). For building swap UIs or integrating the `@txnlab/haystack-router` SDK into an app, load `haystack-router-development` instead.

5. **Prediction markets / Alpha Arcade** — When the user asks about prediction markets, event betting, YES/NO shares, orderbooks, or Alpha Arcade, load the `alpha-arcade-interaction` skill and use the Alpha Arcade MCP tools (`alpha_*`).

## Plugin Capabilities

1. **Algorand Development** — Smart contracts, typed clients, React frontends via AlgoKit CLI and skills
2. **Blockchain Interaction** — Algorand MCP server (121 tools) for direct blockchain access
3. **x402 Payment Protocol** — HTTP-native payments with Algorand as first-class chain
4. **Haystack Router** — DEX aggregator and smart order routing across Algorand DEXes (Tinyman, Pact, Folks) and LST protocols (tALGO, xALGO)
5. **Alpha Arcade** — On-chain prediction markets on Algorand (USDC-denominated, binary/multi-choice)

## Skill Routing

| Capability | Task | Skill |
|------------|------|-------|
| Development | CLI, examples, general workflows | `algorand-development` |
| Development | TypeScript contracts & tools | `algorand-typescript` |
| Development | Python contracts & tools | `algorand-python` |
| Interaction | Blockchain interaction via MCP | `algorand-interaction` |
| x402 | TypeScript x402 development | `algorand-x402-typescript` |
| x402 | Python x402 development | `algorand-x402-python` |
| x402 | Runtime x402 payment (Claude as client) | `algorand-x402-payment` |
| Haystack | SDK integration, React swap UIs, Node.js automation | `haystack-router-development` |
| Haystack | Best-price swaps via MCP tools (agent interaction) | `haystack-router-interaction` |
| Alpha Arcade | Prediction markets via MCP tools | `alpha-arcade-interaction` |

Skills are auto-discovered — Claude invokes them based on task context or via `/skill-name`. Agent `algorand-agent` can be invoked for complex multi-step Algorand tasks.

## MCP Tool Categories (122 tools)

- **Wallet** (10) — `wallet_add_account`, `wallet_remove_account`, `wallet_list_accounts`, `wallet_switch_account`, `wallet_get_info`, `wallet_get_assets`, `wallet_sign_transaction`, `wallet_sign_transaction_group`, `wallet_sign_data`, `wallet_optin_asset`
- **Account Management** (8) — `create_account`, `rekey_account`, `mnemonic_to_mdk`, `mdk_to_mnemonic`, `secret_key_to_mnemonic`, `mnemonic_to_secret_key`, `seed_from_mnemonic`, `mnemonic_from_seed`
- **Utility** (13) — `ping`, `validate_address`, `encode_address`, `decode_address`, `get_application_address`, `bytes_to_bigint`, `bigint_to_bytes`, `encode_uint64`, `decode_uint64`, `verify_bytes`, `sign_bytes`, `encode_obj`, `decode_obj`
- **Transaction** (18) — `make_payment_txn`, `make_keyreg_txn`, `make_asset_create_txn`, `make_asset_config_txn`, `make_asset_destroy_txn`, `make_asset_freeze_txn`, `make_asset_transfer_txn`, `make_app_create_txn`, `make_app_update_txn`, `make_app_delete_txn`, `make_app_optin_txn`, `make_app_closeout_txn`, `make_app_clear_txn`, `make_app_call_txn`, `assign_group_id`, `sign_transaction`, `encode_unsigned_transaction`, `decode_signed_transaction`
- **Algod** (5) — `compile_teal`, `disassemble_teal`, `send_raw_transaction`, `simulate_raw_transactions`, `simulate_transactions`
- **Algod API** (13) — `api_algod_get_account_info`, `api_algod_get_account_application_info`, `api_algod_get_account_asset_info`, `api_algod_get_application_by_id`, `api_algod_get_application_box`, `api_algod_get_application_boxes`, `api_algod_get_asset_by_id`, `api_algod_get_pending_transaction`, `api_algod_get_pending_transactions_by_address`, `api_algod_get_pending_transactions`, `api_algod_get_transaction_params`, `api_algod_get_node_status`, `api_algod_get_node_status_after_block`
- **Indexer API** (17) — `api_indexer_lookup_account_by_id`, `api_indexer_lookup_account_assets`, `api_indexer_lookup_account_app_local_states`, `api_indexer_lookup_account_created_applications`, `api_indexer_search_for_accounts`, `api_indexer_lookup_applications`, `api_indexer_lookup_application_logs`, `api_indexer_search_for_applications`, `api_indexer_lookup_application_box`, `api_indexer_lookup_application_boxes`, `api_indexer_lookup_asset_by_id`, `api_indexer_lookup_asset_balances`, `api_indexer_lookup_asset_transactions`, `api_indexer_search_for_assets`, `api_indexer_lookup_transaction_by_id`, `api_indexer_lookup_account_transactions`, `api_indexer_search_for_transactions`
- **NFDomains** (6) — `api_nfd_get_nfd`, `api_nfd_get_nfds_for_addresses`, `api_nfd_get_nfd_activity`, `api_nfd_get_nfd_analytics`, `api_nfd_browse_nfds`, `api_nfd_search_nfds`
- **Tinyman AMM** (9) — `api_tinyman_get_pool`, `api_tinyman_get_pool_analytics`, `api_tinyman_get_pool_creation_quote`, `api_tinyman_get_liquidity_quote`, `api_tinyman_get_remove_liquidity_quote`, `api_tinyman_get_swap_quote`, `api_tinyman_get_asset_optin_quote`, `api_tinyman_get_validator_optin_quote`, `api_tinyman_get_validator_optout_quote`
- **Haystack Router** (3) — `api_haystack_get_swap_quote`, `api_haystack_execute_swap`, `api_haystack_needs_optin`
- **Pera Asset Verification** (3) — `api_pera_asset_verification_status`, `api_pera_verified_asset_details`, `api_pera_verified_asset_search`
- **Alpha Arcade** (15) — Read: `alpha_get_live_markets`, `alpha_get_reward_markets`, `alpha_get_market`, `alpha_get_orderbook`, `alpha_get_open_orders`, `alpha_get_positions`. Trade: `alpha_create_limit_order`, `alpha_create_market_order`, `alpha_cancel_order`, `alpha_amend_order`, `alpha_propose_match`, `alpha_split_shares`, `alpha_merge_shares`, `alpha_claim`
- **ARC-26 URI** (1) — `generate_algorand_qrcode`
- **Knowledge** (1) — `get_knowledge_doc` (categories: `arcs`, `sdks`, `algokit`, `algokit-utils`, `tealscript`, `puya`, `liquid-auth`, `python`, `developers`, `clis`, `nodes`, `details`)

## Skills vs MCP Tools

- **Skills** = development guidance — how to write contracts, what patterns to follow, ARC standards, project setup. Load a skill when the user wants to *build* something.
- **MCP tools** = direct blockchain interaction — query data, build/sign/send transactions, manage wallet. Call tools when the user wants to *do* something on-chain.
- For development tasks, load the skill first, then use MCP tools as needed during implementation.

## Tool Parameters

- **`network`**: Every blockchain tool accepts `"mainnet"` (default), `"testnet"`, or `"localnet"`. Always confirm with user before mainnet.
- **Pagination**: API tools accept `itemsPerPage` (default 10) and `pageToken` (from previous response) for paging through results.

## Signing

- **`wallet_sign_transaction`** — Signs using the secure OS keychain wallet. Enforces spending limits. **Preferred for all user transactions.**
- **`sign_transaction`** — Signs with a raw secret key hex. Only use when the user explicitly provides their own key.
- **`wallet_sign_transaction_group`** — Signs atomic groups (all-or-nothing) via wallet.

## Common Workflows

**Send ALGO/ASA:**
`wallet_get_info` → `make_payment_txn` or `make_asset_transfer_txn` → `wallet_sign_transaction` → `send_raw_transaction`

**Asset opt-in (shortcut):**
`wallet_optin_asset` (one step — builds, signs, submits)

**Query account:**
`api_algod_get_account_info` (live state) or `api_indexer_lookup_account_by_id` (historical)

**Swap on Tinyman:**
`api_tinyman_get_swap_quote` → build txns from quote → `wallet_sign_transaction_group` → `send_raw_transaction`

**Best-price swap via Haystack Router (DEX aggregator):**
`wallet_get_info` → `api_haystack_needs_optin` (check opt-in) → `api_haystack_get_swap_quote` (preview quote, show user) → user confirms → `api_haystack_execute_swap` (all-in-one: quote + sign + submit). Load skill: `haystack-router-interaction`

> **CRITICAL — Swap direction (`type` parameter):**
> - **"Buy 10 ALGO with USDC"** → user wants **exactly 10 ALGO out** → `type: "fixed-output"`, `amount: 10000000`, `fromASAID: USDC`, `toASAID: ALGO`
> - **"Sell 10 ALGO for USDC"** / **"Swap 10 ALGO to USDC"** → user wants to **spend exactly 10 ALGO** → `type: "fixed-input"`, `amount: 10000000`, `fromASAID: ALGO`, `toASAID: USDC`
> - **"Buy USDC for 10 ALGO"** / **"Use 10 ALGO to buy USDC"** → user specifies **exact input** → `type: "fixed-input"`, `amount: 10000000`, `fromASAID: ALGO`, `toASAID: USDC`
> - Rule: **"buy X of Y"** = fixed-output (exact output). **"swap/sell/use X of Y"** = fixed-input (exact input). Never guess — if ambiguous, ask the user.

**Prediction market trade via Alpha Arcade:**
`wallet_get_info` → `alpha_get_live_markets` (browse markets) → `alpha_get_orderbook` (check liquidity) → `alpha_create_market_order` or `alpha_create_limit_order` (place order). Load skill: `alpha-arcade-interaction`

> **CRITICAL — Alpha Arcade prices are microunits**: `yesProb`/`noProb` range 0–1,000,000 (NOT percentages). $0.50 = 500,000. Orders require both ALGO (MBR ~0.957 per order) and USDC (collateral). Market orders fill immediately and become positions (`alpha_get_positions`), not open orders.

**Atomic group:**
`make_*_txn` (multiple) → `assign_group_id` → `wallet_sign_transaction_group` → `send_raw_transaction`

**x402 payment (Claude as client):**
On HTTP 402 with `accepts[]`: parse requirements → map CAIP-2 → `wallet_get_info` → check opt-in →
`make_payment_txn` (feePayer, amount=0, fee=N×1000 where N=group size [e.g. 2000 for 2 txns], flatFee=true) →
`make_asset_transfer_txn` or `make_payment_txn` (payment, fee=0, flatFee=true) →
`assign_group_id` [feePayer@0, payment@1] → `wallet_sign_transaction` (payment@1 only) →
`encode_unsigned_transaction` (feePayer@0) → construct PAYMENT-SIGNATURE JSON (include `accepted` field) →
`curl -H` retry. Load skill: `algorand-x402-payment`

> **CRITICAL — x402 Base64 blob handling**: When constructing `PAYMENT-SIGNATURE` via shell/curl, NEVER manually copy-paste base64 blobs inline into JSON. Even a single character corruption causes "signature does not match sender" errors. ALWAYS store blobs (`bytes` from `encode_unsigned_transaction`, `blob` from `wallet_sign_transaction`) in separate shell variables first, then interpolate via `${VAR}`. Use `printf '%s'` and `tr -d '\n'` to strip macOS base64 newlines.

## QR Code Display (ARC-26 URI)

When generating QR codes with `generate_algorand_qrcode`, the tool returns: `qr` (UTF-8 text QR code), `uri` (the `algorand://` URI), `link` (shareable hosted QR URL), `expires_in` (link validity period).

**After calling the tool, include all of the following in your response:**

1. **UTF-8 QR block** — Unicode block characters from the `qr` field. Paste inside a code block.
2. **URI string** — from the `uri` field, show for wallet deep links.
3. **Shareable link** — from the `link` field, a hosted QR code URL the user can share.
4. **Expiry** — from `expires_in`, tell user how long the hosted link is valid.

### Steps to include QR code in reply:
1. Call the tool and capture output
2. Extract `qr`, `uri`, `link`, and `expires_in` from the response
3. Include in your reply:

```
[paste UTF-8 QR from `qr` field here]
```

URI: `algorand://...`
Shareable QR: [link URL]
Expires in: [expires_in]

## Post-Transaction Display — Explorer Links

After ANY blockchain operation that produces a transaction ID (txId), **always present the txId to the user as a clickable explorer link**. Choose the URL based on the active network:

- **Mainnet**: `https://allo.info/tx/{txId}`
- **Testnet**: `https://lora.algokit.io/testnet/transaction/{txId}`

Format in your response:
```
Transaction confirmed: [TXID](https://allo.info/tx/TXID)
```

For operations returning multiple transaction IDs (atomic groups, swaps), show **each** txId as a separate clickable link. For `localnet`, show the raw txId without a link (no public explorer).

This applies to results from: `send_raw_transaction`, `wallet_optin_asset`, `api_haystack_execute_swap`, `alpha_create_market_order`, `alpha_create_limit_order`, and any other tool that returns a txId or txIds array.

## Key Things to Remember

- Always check wallet with `wallet_get_info` before blockchain operations
- Use `get_knowledge_doc` for Algorand developer documentation
- Mainnet = real value — always confirm with user before mainnet transactions
- Default to testnet during development
- Every transaction costs 0.001 ALGO minimum
- Account needs 0.1 ALGO base + 0.1 per asset/app opt-in (MBR)

## Common Mainnet Assets

| Asset | ASA ID | Decimals |
|-------|--------|----------|
| ALGO | native (0) | 6 |
| USDC | 31566704 | 6 |
| USDT | 312769 | 6 |
| goETH | 386192725 | 8 |
| goBTC | 386195940 | 8 |

## Important patterns

- **NEVER use PyTEAL or Beaker** — these are legacy. Use Algorand TypeScript or Algorand Python.
- **NEVER use AlgoExplorer** — obsolete. Use Allo.info for block/account/transaction data.
- **NFD (.algo names)**: Always use `depositAccount` field for transactions.
- **Asset verification**: Before transacting unknown ASAs, check `api_pera_asset_verification_status` — tiers: `verified`, `trusted`, `suspicious`, `unverified`. Warn user about suspicious/unverified assets.
- **Alpha Arcade prices are microunits**: `yesProb`/`noProb` range 0–1,000,000 (NOT percentages). $0.50 = 500,000. Orders require both ALGO (MBR) and USDC (collateral).

## External resources

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
- GoPlausible x402-avm Documentation and Example code : https://github.com/GoPlausible/.github/blob/main/profile/algorand-x402-documentation/README.md
- GoPlausible x402-avm Examples template Projects : https://github.com/GoPlausible/x402-avm/tree/branch-v2-algorand-publish/examples/
- CAIP-2 Specification : https://github.com/ChainAgnostic/CAIPs/blob/main/CAIPs/caip-2.md
- Coinbase x402 Protocol : https://github.com/coinbase/x402
- Haystack Router (TxnLab DEX Aggregator) : https://github.com/TxnLab/haystack-router
- Alpha Arcade: https://alphaarcade.com
- Alpha Arcade API: https://platform.alphaarcade.com
