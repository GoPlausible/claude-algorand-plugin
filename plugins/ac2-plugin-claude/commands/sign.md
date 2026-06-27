---
description: Ask the connected wallet to sign data (message / transaction / bytes)
argument-hint: [what to sign]
---

The user wants the connected AC2 wallet to sign: $ARGUMENTS

Run the AC2 signing flow:

1. If you don't already know the wallet's accounts/identities, call
   `ac2_capabilities` first to choose the correct signer and chain.
2. Call `ac2_sign` with the appropriate payload, key type, and sig hint for what
   the user asked to sign.
3. The user approves the request in their wallet (passkey/biometric). Relay the
   returned signature — and which account/identity signed — back to the user.

Never fabricate a signature; it must come back from the wallet. If nothing is
paired, tell the user to run `/ac2:pair` first.
