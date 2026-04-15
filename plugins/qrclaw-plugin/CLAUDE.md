# QRClaw Plugin

This plugin provides **QRClaw** — an open source edge service (qrclaw.goplausible.xyz) that generates QR codes from any string and returns both a shareable smart link (with rich social previews) and a UTF-8 text QR that renders directly in terminals.

## Required — Load Skill First

**Before generating any QR code, ALWAYS load and read the `qrclaw` skill first.** It defines the security rules, API contract, channel-aware display formatting, and rate limits needed to use the service correctly.

```
/qrclaw-plugin:qrclaw
```

Do NOT call the QRClaw endpoint without loading the skill first — it contains the mandatory sensitive-data checks and the JSON-vs-redirect header requirement.

## Service

The plugin targets a single public GET endpoint:

```
https://qrclaw.goplausible.xyz/?q=<url-encoded-string>
```

- Always send `Accept: application/json` — without it the endpoint returns a 302 redirect to the HTML page instead of JSON.
- Response fields: `link` (smart link URL), `qr` (UTF-8 block QR), `data` (original input), `expires_in` (`"24h"`).
- No auth required. Source: [github.com/GoPlausible/qrclaw](https://github.com/GoPlausible/qrclaw).

## Key Rules

- **Never send sensitive data.** Refuse requests for QR codes containing private keys, mnemonics, passwords, API tokens, JWTs, session IDs, or PII. The service transmits and temporarily stores input and exposes it via a public smart link.
- **Rate limit: 5 QR codes per minute per IP.** Do not generate QR codes in tight loops — batch or delay.
- **URL-encode the `q` parameter** — special characters, spaces, and URI schemes like `algorand://` must be encoded.
- **Channel-aware output:** in TUI/web contexts show the full UTF-8 QR block plus data plus smart link; in messaging platforms (Telegram, Discord, Slack, etc.) skip the UTF-8 block and show only the data and smart link, which renders as a rich Open Graph preview.
- **Expiry:** all QR codes and smart links expire after 24 hours.
