# Algorand MCP — Workflow Examples

> All examples use `network: "testnet"` unless noted. For mainnet, change to `"mainnet"` and confirm with user.
> USDC ASA ID: 31566704 (6 decimals). 1 ALGO = 1,000,000 microAlgos.

---

## Session Start

### Step 1: Verify wallet
```
wallet_get_info { "network": "testnet" }
```

### Step 2: If no account exists
```
wallet_add_account {
  "nickname": "my-wallet"
}
```

### Step 3: If account needs funding
```
generate_algorand_qrcode {
  "address": "[wallet_address]",
  "amount": 5000000,
  "note": "Fund testnet account"
}
```
Or direct user to: https://lora.algokit.io/testnet/fund

### Step 4: If account needs USDC funding
```
generate_algorand_qrcode {
  "address": "[wallet_address]",
  "asset": 10458941, // USDC on testnet
  "amount": 1000000, // 1 USDC with 6 decimals
  "note": "Fund testnet account with USDC"
}
```
Or direct user to: https://faucet.circle.com/

---

## ALGO Payment Workflow

### Step 1: Get wallet info
```
wallet_get_info { "network": "testnet" }
```

### Step 2: Create payment transaction
```
make_payment_txn {
  "from": "[sender_address]",
  "to": "[receiver_address]",
  "amount": 1000000,
  "note": "Payment of 1 ALGO",
  "network": "testnet"
}
```
> Amount in microAlgos: 1 ALGO = 1,000,000

### Step 3: Sign the transaction
```
wallet_sign_transaction {
  "transaction": { "...transaction object from step 2..." },
  "network": "testnet"
}
```

### Step 4: Submit the transaction
```
send_raw_transaction {
  "signedTxns": ["[base64_blob_from_step_3]"],
  "network": "testnet"
}
```

### Step 5: Verify (optional)
```
api_indexer_lookup_transaction_by_id {
  "txId": "[txID_from_step_3]",
  "network": "testnet"
}
```

### Step 6: Present transaction link to user
Show the txId from Step 4 as a clickable explorer link:
- Testnet: `https://lora.algokit.io/testnet/transaction/[txID]`
- Mainnet: `https://allo.info/tx/[txID]`

---

## Asset Opt-In Workflow (One-Step)

The simplest way to opt-in to an asset:

```
wallet_optin_asset {
  "assetId": 31566704,
  "network": "testnet"
}
```

This creates, signs, and submits the opt-in transaction in a single call.

Present the returned txId as a clickable explorer link:
- Testnet: `https://lora.algokit.io/testnet/transaction/[txID]`
- Mainnet: `https://allo.info/tx/[txID]`

---

## Asset Opt-In Workflow (Manual)

### Step 1: Check if already opted in
```
api_algod_get_account_asset_info {
  "address": "[sender_address]",
  "assetId": 31566704,
  "network": "testnet"
}
```

### Step 2: Create opt-in transaction (0-amount self-transfer)
```
make_asset_transfer_txn {
  "from": "[sender_address]",
  "to": "[sender_address]",
  "assetIndex": 31566704,
  "amount": 0,
  "network": "testnet"
}
```

### Step 3: Sign
```
wallet_sign_transaction {
  "transaction": { "...transaction from step 2..." },
  "network": "testnet"
}
```

### Step 4: Submit
```
send_raw_transaction {
  "signedTxns": ["[base64_blob_from_step_3]"],
  "network": "testnet"
}
```

### Step 5: Present transaction link to user
Show the txId as a clickable explorer link (testnet: `https://lora.algokit.io/testnet/transaction/[txID]`, mainnet: `https://allo.info/tx/[txID]`).

---

## Asset Transfer Workflow (USDC Example)

### Step 1: Get wallet info
```
wallet_get_info { "network": "testnet" }
```

### Step 2: Get asset info (verify decimals)
```
api_algod_get_asset_by_id {
  "assetId": 31566704,
  "network": "testnet"
}
```

### Step 3: Check sender's asset balance
```
api_algod_get_account_asset_info {
  "address": "[sender_address]",
  "assetId": 31566704,
  "network": "testnet"
}
```

### Step 4: Verify recipient has opted in
```
api_algod_get_account_asset_info {
  "address": "[recipient_address]",
  "assetId": 31566704,
  "network": "testnet"
}
```

### Step 5: Create transfer (1 USDC = 1,000,000 with 6 decimals)
```
make_asset_transfer_txn {
  "from": "[sender_address]",
  "to": "[recipient_address]",
  "assetIndex": 31566704,
  "amount": 1000000,
  "network": "testnet"
}
```

### Step 6: Sign
```
wallet_sign_transaction {
  "transaction": { "...transaction from step 5..." },
  "network": "testnet"
}
```

### Step 7: Submit
```
send_raw_transaction {
  "signedTxns": ["[base64_blob_from_step_6]"],
  "network": "testnet"
}
```

### Step 8: Present transaction link to user
Show the txId as a clickable explorer link (testnet: `https://lora.algokit.io/testnet/transaction/[txID]`, mainnet: `https://allo.info/tx/[txID]`).

---

## Atomic Group Transaction Workflow

### Step 1: Build individual transactions
```
make_payment_txn {
  "from": "[address_A]",
  "to": "[address_B]",
  "amount": 1000000,
  "network": "testnet"
}
```
```
make_asset_transfer_txn {
  "from": "[address_B]",
  "to": "[address_A]",
  "assetIndex": 31566704,
  "amount": 500000,
  "network": "testnet"
}
```

### Step 2: Assign group ID
```
assign_group_id {
  "transactions": [txn1_from_step1, txn2_from_step1]
}
```

### Step 3: Sign group with wallet
```
wallet_sign_transaction_group {
  "transactions": [grouped_txn1, grouped_txn2],
  "network": "testnet"
}
```

### Step 4: Submit
```
send_raw_transaction {
  "signedTxns": ["[blob1]", "[blob2]"],
  "network": "testnet"
}
```

> Atomic groups are all-or-nothing: either all transactions succeed or none do.

### Step 5: Present transaction links to user
Show each txId from the group as a clickable explorer link (testnet: `https://lora.algokit.io/testnet/transaction/[txID]`, mainnet: `https://allo.info/tx/[txID]`).

---

## Create an ASA (Algorand Standard Asset)

### Step 1: Create the asset
```
make_asset_create_txn {
  "from": "[creator_address]",
  "total": 1000000000,
  "decimals": 6,
  "defaultFrozen": false,
  "unitName": "MYT",
  "assetName": "My Token",
  "assetURL": "https://example.com/my-token",
  "manager": "[creator_address]",
  "reserve": "[creator_address]",
  "freeze": "[creator_address]",
  "clawback": "[creator_address]",
  "network": "testnet"
}
```

### Step 2: Sign
```
wallet_sign_transaction {
  "transaction": { "...transaction from step 1..." },
  "network": "testnet"
}
```

### Step 3: Submit
```
send_raw_transaction {
  "signedTxns": ["[base64_blob]"],
  "network": "testnet"
}
```

### Step 4: Look up the created asset
```
api_algod_get_pending_transaction {
  "txId": "[txID_from_step_2]",
  "network": "testnet"
}
```
> The `asset-index` field in the response contains the new ASA ID.

Present the txId and new ASA ID to the user. Show the txId as a clickable explorer link (testnet: `https://lora.algokit.io/testnet/transaction/[txID]`, mainnet: `https://allo.info/tx/[txID]`).

---

## Deploy a Smart Contract

### Step 1: Compile approval program
```
compile_teal {
  "source": "#pragma version 10\nint 1\nreturn",
  "network": "testnet"
}
```

### Step 2: Compile clear program
```
compile_teal {
  "source": "#pragma version 10\nint 1\nreturn",
  "network": "testnet"
}
```

### Step 3: Create the application
```
make_app_create_txn {
  "from": "[creator_address]",
  "approvalProgram": "[base64_from_step_1]",
  "clearProgram": "[base64_from_step_2]",
  "numGlobalByteSlices": 1,
  "numGlobalInts": 1,
  "numLocalByteSlices": 0,
  "numLocalInts": 0,
  "network": "testnet"
}
```

### Step 4: Sign and submit
```
wallet_sign_transaction {
  "transaction": { "...transaction from step 3..." },
  "network": "testnet"
}
```
```
send_raw_transaction {
  "signedTxns": ["[base64_blob]"],
  "network": "testnet"
}
```

### Step 5: Present transaction link to user
Show the txId as a clickable explorer link (testnet: `https://lora.algokit.io/testnet/transaction/[txID]`, mainnet: `https://allo.info/tx/[txID]`).

---

## NFD Lookup

### Look up an NFD name
```
api_nfd_get_nfd {
  "nameOrID": "example.algo",
  "view": "full",
  "network": "mainnet"
}
```

### Get NFDs for an address
```
api_nfd_get_nfds_for_addresses {
  "address": ["ALGO_ADDRESS"],
  "view": "brief",
  "network": "mainnet"
}
```

> **CRITICAL**: When sending to an NFD, always use the `depositAccount` field from the response, NOT other address fields.

---

## Tinyman Swap Quote

### Get a swap quote (ALGO → USDC)
```
api_tinyman_get_swap_quote {
  "asset1Id": 0,
  "asset2Id": 31566704,
  "amount": 1000000,
  "network": "mainnet"
}
```
> Asset ID 0 = ALGO

### Get pool info
```
api_tinyman_get_pool {
  "asset1Id": 0,
  "asset2Id": 31566704,
  "version": "v2",
  "network": "mainnet"
}
```

### Get pool analytics (volume, TVL, fees)
```
api_tinyman_get_pool_analytics {
  "asset1Id": 0,
  "asset2Id": 31566704,
  "network": "mainnet"
}
```

> For more Tinyman workflows (liquidity, pool creation, validator opt-in/out), use the Tinyman tools directly. For best-price swaps across multiple DEXes, use Haystack Router below.

---

## Haystack Router — Best-Price Swap (DEX Aggregator)

Haystack Router aggregates quotes across Tinyman, Pact, Folks, and LST protocols to find the optimal swap route. For detailed configuration and batch workflows, load the `haystack-router-interaction` skill.

### CRITICAL: Swap Direction (`type` parameter)

The `type` parameter controls whether `amount` is exact input or exact output. **Wrong direction = wrong amount spent/received.**

| User says | `type` | `amount` is | `fromASAID` | `toASAID` |
|-----------|--------|-------------|-------------|-----------|
| "Buy 10 ALGO with USDC" | `"fixed-output"` | 10000000 (desired output) | USDC | ALGO |
| "Sell 10 ALGO for USDC" | `"fixed-input"` | 10000000 (exact spend) | ALGO | USDC |
| "Swap 10 ALGO to USDC" | `"fixed-input"` | 10000000 (exact spend) | ALGO | USDC |
| "Use 10 ALGO to buy USDC" | `"fixed-input"` | 10000000 (exact spend) | ALGO | USDC |
| "Buy USDC for 10 ALGO" | `"fixed-input"` | 10000000 (exact spend) | ALGO | USDC |

**Rule: "buy X of Y" = fixed-output. "sell/swap/use X of Y" = fixed-input. If ambiguous, ask.**

### Example: fixed-input ("Swap 1 ALGO to USDC" — spend exactly 1 ALGO)

#### Step 1: Check wallet
```
wallet_get_info { "network": "testnet" }
```

#### Step 2: Execute swap
```
api_haystack_execute_swap {
  "fromASAID": 0,
  "toASAID": 31566704,
  "amount": 1000000,
  "type": "fixed-input",
  "slippage": 0.5,
  "network": "testnet"
}
```
> User spends exactly 1 ALGO. Output USDC varies based on market price.

### Example: fixed-output ("Buy 1 USDC with ALGO" — receive exactly 1 USDC)

```
api_haystack_execute_swap {
  "fromASAID": 0,
  "toASAID": 31566704,
  "amount": 1000000,
  "type": "fixed-output",
  "slippage": 0.5,
  "network": "testnet"
}
```
> User receives exactly 1 USDC. Input ALGO varies based on market price.

After execution, present each txId from the response as a clickable explorer link (testnet: `https://lora.algokit.io/testnet/transaction/[txID]`, mainnet: `https://allo.info/tx/[txID]`).

### Recommended workflow with preview (4 steps)

#### Step 1: Check wallet
```
wallet_get_info { "network": "testnet" }
```

#### Step 2: Check if opt-in is needed
```
api_haystack_needs_optin {
  "address": "[wallet_address]",
  "assetId": 31566704,
  "network": "testnet"
}
```
If `needsOptIn` is true:
```
wallet_optin_asset { "assetId": 31566704, "network": "testnet" }
```

#### Step 3: Preview quote (show user before executing)
```
api_haystack_get_swap_quote {
  "fromASAID": 0,
  "toASAID": 31566704,
  "amount": 1000000,
  "type": "fixed-input",
  "address": "[wallet_address]",
  "network": "testnet"
}
```
> Show user: expected output, USD values, route, price impact. **Always confirm before executing.**
> Set `type` based on user intent — see direction table above.

#### Step 4: Execute after user confirms
```
api_haystack_execute_swap {
  "fromASAID": 0,
  "toASAID": 31566704,
  "amount": 1000000,
  "type": "fixed-input",
  "slippage": 0.5,
  "network": "testnet"
}
```

#### Step 5: Present transaction links to user
`api_haystack_execute_swap` returns `txIds` (an array). Show each txId as a clickable explorer link:
- Testnet: `https://lora.algokit.io/testnet/transaction/[txID]`
- Mainnet: `https://allo.info/tx/[txID]`

### Slippage guidance

| Pair Type | Recommended Slippage |
|-----------|---------------------|
| ALGO ↔ USDC (deep liquidity) | 0.1–0.5% |
| Major ASAs | 0.5–1% |
| Low-liquidity pairs | 1–3% |

---

## Pera Asset Verification

Verify asset legitimacy before transacting. Mainnet only — uses Pera Wallet's verification database.

### Check if an asset is verified
```
api_pera_asset_verification_status {
  "assetId": 31566704
}
```
> Returns verification tier: `verified`, `trusted`, `suspicious`, or `unverified`. Warn user about suspicious/unverified assets before proceeding.

### Get detailed asset info (name, USD value, logo, supply)
```
api_pera_verified_asset_details {
  "assetId": 31566704
}
```

### Search for verified assets by name
```
api_pera_verified_asset_search {
  "query": "USDC",
  "verifiedOnly": true
}
```
> Set `verifiedOnly: true` to filter out suspicious and unverified assets.

---

## Using the Knowledge Base

### Get a specific document
```
get_knowledge_doc {
  "documents": ["arcs:specs:arc-0003.md"]
}
```

### Knowledge categories
- `arcs` — Algorand Request for Comments
- `sdks` — Software Development Kits
- `algokit` — AlgoKit
- `algokit-utils` — AlgoKit Utils
- `tealscript` — TEALScript
- `puya` — Puya compiler
- `liquid-auth` — Liquid Auth
- `python` — Python Development
- `developers` — Developer Documentation
- `clis` — CLI Tools
- `nodes` — Node Management
- `details` — Developer Details

---

## Compile and Disassemble TEAL

### Compile TEAL
```
compile_teal {
  "source": "#pragma version 10\nint 1\nreturn",
  "network": "testnet"
}
```

### Disassemble TEAL bytecode
```
disassemble_teal {
  "bytecode": "[base64_bytecode_from_compile]",
  "network": "testnet"
}
```

---

## Encode/Decode Objects (msgpack)

### Encode to msgpack
```
encode_obj {
  "obj": { "key": "value", "num": 42 }
}
```

### Decode from msgpack
```
decode_obj {
  "bytes": "[base64_msgpack_string]"
}
```

---

## Using External Keys (Non-Wallet Signing)

When a user provides their own secret key instead of using the wallet:

### Step 1: Build transaction
```
make_payment_txn {
  "from": "[sender_address]",
  "to": "[receiver_address]",
  "amount": 1000000,
  "network": "testnet"
}
```

### Step 2: Sign with external key
```
sign_transaction {
  "transaction": { "...transaction from step 1..." },
  "sk": "[hex_encoded_secret_key]"
}
```

### Step 3: Submit
```
send_raw_transaction {
  "signedTxns": ["[base64_blob_from_step_2]"],
  "network": "testnet"
}
```

### Step 4: Present transaction link to user
Show the txId as a clickable explorer link (testnet: `https://lora.algokit.io/testnet/transaction/[txID]`, mainnet: `https://allo.info/tx/[txID]`).

---

## Top-Up QR Code (Insufficient Funds)

When balance is insufficient, generate an ARC-26 QR code for easy funding:

```
generate_algorand_qrcode {
  "address": "[wallet_address]",
  "amount": 5000000,
  "note": "Fund account for transaction"
}
```

The response includes `qr` (UTF-8 text QR code), `uri` (the `algorand://` URI), `link` (shareable hosted QR URL), and `expires_in` (link validity period). The QR code can be scanned with any Algorand-compatible wallet, and the hosted link can be shared directly.

---

## Simulate Before Submitting

### Simulate a transaction to check for errors
```
simulate_transactions {
  "txnGroups": [ "...transaction group..." ],
  "allowEmptySignatures": true,
  "allowMoreLogging": true,
  "network": "testnet"
}
```

This lets you verify a transaction will succeed before actually submitting it.


---

## x402 Payments & Bazaar Discovery — see the dedicated skill

For workflow examples covering paid HTTP resources (HTTP 402), the `x402_discover_payment_requirements` and `make_http_request_with_x402` tools, and Bazaar discovery (`bazaar_list`, `bazaar_search`, `bazaar_get_resource_details`), **load the `algorand-x402-payment` skill**. It contains:

- The three payment patterns with full input/output examples
- Tool argument tables (including all client-side post-filters on `bazaar_search`)
- Response shape examples
- Common errors with recovery steps
- Worked end-to-end testnet weather example

The dedicated skill is the source of truth for x402 + Bazaar workflows. The examples in this file are for the rest of the algorand-mcp tool surface (wallet, transactions, ASA, AMM, NFD, etc.).
