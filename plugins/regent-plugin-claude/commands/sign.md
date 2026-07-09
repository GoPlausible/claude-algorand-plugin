---
description: Ask the connected wallet to sign data (message / transaction / bytes)
argument-hint: [what to sign]
---

The user wants the connected Regent wallet to sign: $ARGUMENTS

Run the Regent signing flow:

1. If you don't already know the wallet's accounts/identities, call
   `regent_capabilities` first to choose the correct signer and chain.
2. Call `regent_sign` with the appropriate payload, key type, and sig hint for what
   the user asked to sign.
3. The user approves the request in their wallet (passkey/biometric). Relay the
   returned signature — and which account/identity signed — back to the user.

Never fabricate a signature; it must come back from the wallet. If nothing is
paired, tell the user to run `/regent:pair` first.
