---
description: Unpair the connected AC2 wallet (destructive)
---

Unpair the currently connected AC2 wallet by calling the `ac2_forget` tool.

This is destructive — it drops the pairing and the session will have to pair
again (via `/ac2:pair`) to reconnect. Unless the user has clearly asked to
unpair/forget, confirm with them before calling the tool.
