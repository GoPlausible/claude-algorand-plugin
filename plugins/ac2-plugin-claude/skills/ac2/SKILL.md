---
name: ac2
description: "How to use the AC2 channel to get cryptographic signatures from the user's connected wallet (e.g. Regent) over a live WebRTC link. Use this whenever you need to sign or verify something on behalf of the user — wallet sign-in, an Algorand/EVM/Solana message or transaction, an off-chain attestation/VC, or whenever the user asks you to 'sign', 'approve', or 'authorize' with their wallet, even if they don't say 'AC2'. The agent never holds keys; the wallet does."
---

# AC2 — remote wallet signing

AC2 connects you to the **user's wallet** (such as Regent) over a live WebRTC data channel. The user is the human on the other end of the AC2 connection. The wallet is the key custodian: **you never see or hold private keys.** You ask the wallet to sign; the user approves each request in-wallet with a passkey or biometric, then the signature comes back to you.

You interact through one discovery tool, one signing tool, and a matching set of verifier tools (all prefixed `ac2_`).

## Discover first: `ac2_capabilities`

**Call `ac2_capabilities` at the start of every new conversation.** Every call hits the wire and returns the wallet's current state — the cache was removed in plugin v0.0.84 so you don't have to worry about stale primaries / accounts after the user changes wallet state. (The tool still accepts a `refresh` parameter for API stability but it's a no-op.)

The tool returns:

- **Wallet descriptor**: name, version, protocol, supported features.
- **Identities**: the signing identities (public keys / DIDs) the user has provisioned. Primary identities carry a `bip_method` field (e.g. `BIP32-Ed25519`).
- **Accounts grouped by chain**: e.g. `algorand: [ADDR…]`, `base: [0x…]`, `solana: [...]`. These are the **real signing addresses** for derived (transactable) accounts — don't guess.
- **Primary accounts grouped by BIP derivation method**: e.g. `BIP32-Ed25519: [<pubkey>…]`, `SLIP10-Ed25519: [<pubkey>…]`. Primaries are identity-only roots (no transactable address); they're surfaced by derivation method rather than chain because a primary key is chain-agnostic at the user-visible layer.
- **This agent's own descriptor**: version + the `sig_hint`s this plugin can verify.

Use the result to:

1. Pick a `sig_hint` the wallet **actually** supports — not just one the protocol catalog lists. The wallet implements a subset. Each identity's `bip_method` tells you that key's curve (`*-Ed25519` → that key signs `raw-ed25519`, not `raw-secp256k1` — different curves, a cryptographic fact, not a setting); the chains under `accounts` tell you which `message-*`/`transaction-*` hints exist. A `raw-*` hint is available only if **some** identity in the list has the matching curve, so scan all identities before concluding. Don't request `transaction-solana` with no Solana account, or `raw-secp256k1` when every identity is Ed25519. See [Which of these can my wallet actually do?](#which-of-these-can-my-wallet-actually-do).
2. Reference real addresses in the `description` (e.g. `"Sign in as <did>"`, `"Send 5 USDC from <base addr>"`) — never invented placeholders.
3. Detect "no live session" early and prompt the user to connect their wallet instead of failing in `ac2_sign`.

Call again any time you want fresh state — there's no penalty for re-calling within a conversation. WebRTC round-trip on the local data channel is sub-millisecond.

## The core loop: sign → verify

1. Call **`ac2_sign`** with the bytes to sign, a human-readable `description`, and the right `sig_hint`.
2. The wallet shows the user an approval modal; they pick the account/identity and confirm.
3. On approval you get back a base64 `signature` plus the signer's `public_key` (or EVM `address`).
4. Optionally call the **matching verifier** to confirm the signature locally (pure, no round-trip).

`ac2_sign` returns `{ status: "rejected", reason }` if the user declines — treat that as a normal outcome, not an error, and tell the user what was declined.

## Pick the right `sig_hint` (this is the part that goes wrong)

`sig_hint` selects the curve, prefix, and encoding the wallet applies. Each one has exactly one verifier: **`ac2_verify_<sig_hint with - → _>`**.

> **This table is the protocol catalog — the full set of hints AC2 *defines* and this agent can *verify*. It is NOT a list of what the connected wallet supports.** Wallets implement a subset. A row appearing here does not mean your wallet can produce that signature. The wallet's `ac2_capabilities` response is the only authority on what's actually available — see [Which of these can my wallet do?](#which-of-these-can-my-wallet-actually-do) below. Requesting a hint the wallet can't fulfil gets rejected.

| `sig_hint` | `key_type` | Chain / use | Verifier tool |
|---|---|---|---|
| `raw-ed25519` | identity | Algorand/Solana identity, off-chain attestation/VC | `ac2_verify_raw_ed25519` |
| `raw-secp256k1` | identity | Base/EVM identity, off-chain | `ac2_verify_raw_secp256k1` |
| `message-algorand` | account | Algorand signed message (ARC-60) | `ac2_verify_message_algorand` |
| `message-evm` | account | EVM `personal_sign` (EIP-191) | `ac2_verify_message_evm` |
| `message-solana` | account | Solana signed message | `ac2_verify_message_solana` |
| `typed-data-evm` | account | EVM EIP-712 typed data | `ac2_verify_typed_data_evm` |
| `transaction-algorand` | account | Sign an Algorand transaction | `ac2_verify_transaction_algorand` |
| `transaction-evm` | account | Sign an EVM transaction | `ac2_verify_transaction_evm` |
| `transaction-solana` | account | Sign a Solana transaction | `ac2_verify_transaction_solana` |

**`key_type` must match the hint:** `identity` → `raw-*` only (DID-bound key, never holds funds — for sign-in, attestations, VCs, AP2 mandates). `account` → `message-*` / `typed-data-evm` / `transaction-*` (on-chain account key). Defaults to `account`.

Omitting `sig_hint` falls back to plain Ed25519-over-raw-bytes (works only for an Algorand identity key; anything else is rejected with `sig_hint_required`). **Always set `sig_hint` explicitly** so the wallet applies the correct chain encoding.

### Which of these can my wallet actually do?

The capabilities response does **not** hand you a ready-made list of supported `sig_hint`s. You derive it from the wallet's identities and accounts. Match the request to what's present, not to what the table above lists:

- **Identity hints (`raw-*`) are gated by the curve of each individual identity key**, read from that identity's `bip_method`. Curves don't cross — an Ed25519 private key cannot produce a secp256k1 signature and vice versa; this is math, not a wallet setting:
  - An identity with `BIP32-Ed25519` / `SLIP10-Ed25519` (Ed25519 curve) can sign `raw-ed25519`, **not `raw-secp256k1`**.
  - An identity with a secp256k1 derivation method can sign `raw-secp256k1`, not `raw-ed25519`.
  - A wallet may list several identities of **different** curves, so evaluate per hint across the whole list: `raw-ed25519` is available if *any* identity is Ed25519; `raw-secp256k1` if *any* identity is secp256k1. Only when **every** identity is Ed25519 (e.g. the list is just `[BIP32-Ed25519]`) is `raw-secp256k1` genuinely unavailable — and then it's rejected no matter what the protocol catalog lists.
- **Account hints (`message-*` / `typed-data-evm` / `transaction-*`) are gated by the chains under `accounts`:**
  - `algorand` accounts → `message-algorand`, `transaction-algorand`.
  - `base`/EVM accounts → `message-evm`, `typed-data-evm`, `transaction-evm`.
  - `solana` accounts → `message-solana`, `transaction-solana`.
  - No accounts for a chain → none of that chain's hints are available; don't request them.

When in doubt, pick the hint whose curve/chain you can point to a concrete identity or address for in the `ac2_capabilities` output. If the user asks for something the wallet can't sign (e.g. an EVM identity signature on an Ed25519-only wallet), say so and tell them what their wallet *can* do — don't fire a request you already know will be rejected.

## Payloads

- `payload_base64` is the **raw** bytes — the wallet adds the chain prefix/encoding itself based on `sig_hint`. Don't pre-prefix.
- For `transaction-*`, pass the **canonical unsigned-transaction bytes** (Algorand msgpack / EVM RLP / Solana serialized). The wallet returns the signed form.
- `display_hint` (`text` | `json` | `hex`) only controls how the modal *renders* the payload — no cryptographic effect. Set it when the auto-detected preview would be unhelpful.

## Good `description` values

The `description` is the only thing the user reads before approving — make it specific and honest. Examples: `"Sign in to BankApp as alice@example.com"`, `"Approve transfer of 5 USDC to 0x1234…abcd"`, `"Issue an AP2 payment mandate for $20/mo"`. Vague descriptions like `"sign this"` get declined.

## If there's no connection

These tools require a live AC2 session (the wallet paired and connected). If `ac2_sign` reports no active session, call `ac2_pair` to get a QR code / deep link, show it to the user so they can scan it with their wallet, and check `ac2_status` — don't retry signing in a loop.

## Paying for HTTP resources (x402)

Some HTTP endpoints require a small on-chain payment to access (the **x402** protocol). You can pay for them with the **user's wallet** — you never hold funds; the user approves and signs each payment in-wallet. The tools are all `_ac2`-suffixed (so they never clash with a separate local x402 MCP):

- `x402_discover_payment_requirements_ac2` — probe an endpoint and see what it costs, **without** paying.
- `make_http_request_with_x402_ac2` — call the endpoint and pay automatically from the wallet. It builds the payment; the user approves + signs in their wallet (e.g. Regent shows the amount, asset, and recipient); then the request is retried with the payment attached.
- `bazaar_list_ac2` / `bazaar_search_ac2` / `bazaar_get_resource_details_ac2` — browse/search the Bazaar directory of paid API resources (no payment, no wallet needed).

**Before paying:** call `ac2_capabilities` and confirm the wallet has an **Algorand account** (under `accounts.algorand`). The payment comes from that account.

**Choosing the paying account (important):**
- If the wallet has **exactly one** Algorand account, `make_http_request_with_x402_ac2` uses it automatically — pass nothing extra.
- If the wallet has **more than one** Algorand account, the tool does NOT guess: it returns `needsPayerSelection: true` with the account list instead of paying. **Ask the user which account to pay from**, then call the tool again with `payerAddress` set to their choice.

**Budget:** pass `maxAmountPerRequest` (USDC atomic units; 1,000,000 = $1.00) to cap spend. If a requirement exceeds it, the call refuses rather than over-paying.

If the tool reports no Algorand account or no live session, tell the user to connect/pair their wallet (or add an Algorand account) — don't retry in a loop.

**ALWAYS confirm the payment with an explorer link.** After a successful `make_http_request_with_x402_ac2`, find the on-chain transaction id in the tool's response (it's in the settlement info — typically `paymentResponse`, e.g. a `txId`/`transaction` field; the response also carries `paid.network`) and reply to the user with a clickable explorer link to that transaction, chosen by network:

- **testnet** → `https://lora.algokit.io/testnet/transaction/<TXID>` (Lora)
- **mainnet** → `https://allo.info/tx/<TXID>` (allo.info)

Example: `Paid 0.5 USDC ✓ — https://lora.algokit.io/testnet/transaction/PH4OF7FW5DIRRFFHSFMN5X722UFUQEA4G4VDUVYVL3LFXZ5RO4SQ`. Always include this link so the user can verify the payment on-chain.
