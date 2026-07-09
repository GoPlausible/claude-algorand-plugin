---
description: Unpair the connected Regent wallet (destructive)
---

Unpair the currently connected Regent wallet by calling the `regent_forget` tool.

This is destructive — it drops the pairing and the session will have to pair
again (via `/regent:pair`) to reconnect. Unless the user has clearly asked to
unpair/forget, confirm with them before calling the tool.
