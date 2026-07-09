---
description: List the available Regent commands
---

Tell the user the available Regent slash commands and what each does:

- `/regent:pair` — pair a wallet (shows a QR + deep link to scan with Regent)
- `/regent:status` — connection status (read-only)
- `/regent:capabilities` — list the wallet's identities and per-chain accounts
- `/regent:sign <what>` — ask the wallet to sign a message / transaction / bytes
- `/regent:version` — plugin + bundled SDK versions
- `/regent:forget` — unpair the wallet (destructive)

Also mention they can just talk normally ("pair my wallet", "sign this message")
and you'll use the underlying `regent_*` tools directly.
