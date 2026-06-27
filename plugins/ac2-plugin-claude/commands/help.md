---
description: List the available AC2 commands
---

Tell the user the available AC2 slash commands and what each does:

- `/ac2:pair` — pair a wallet (shows a QR + deep link to scan with Regent)
- `/ac2:status` — connection status (read-only)
- `/ac2:capabilities` — list the wallet's identities and per-chain accounts
- `/ac2:sign <what>` — ask the wallet to sign a message / transaction / bytes
- `/ac2:version` — plugin + bundled SDK versions
- `/ac2:forget` — unpair the wallet (destructive)

Also mention they can just talk normally ("pair my wallet", "sign this message")
and you'll use the underlying `ac2_*` tools directly.
