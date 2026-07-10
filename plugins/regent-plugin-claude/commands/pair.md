---
description: Pair a Regent wallet (e.g. Regent) — shows a QR code + deep link to scan
---

Start Regent wallet pairing by calling the `regent_pair` tool.

Print the returned QR code and Liquid Auth deep link to the user EXACTLY as the
tool returns them — do not summarize, redraw, or truncate the QR. Tell the user
to scan the QR with their wallet app (Regent) or open the deep link on their
phone, then wait for the connection to come up. If a pairing is already active,
`regent_pair` refreshes it.
