---
name: temporam
description: Use Temporam temporary email — generate inbound addresses, wait for messages (OTP/verification), and optionally create sender mailboxes to send mail via API or MCP.
---

# Temporam

Temporary email **v3** API. Auth: `Authorization: Bearer <API_KEY>`. Base URL: `https://api.temporam.com`. Paths are under `/v3` only. Do not call `/v1`.

Do not invent endpoints. Prefer MCP tools when `@temporam/mcp` is configured.

## MCP

```json
{
  "mcpServers": {
    "temporam": {
      "command": "npx",
      "args": ["-y", "@temporam/mcp"],
      "env": {
        "TEMPORAM_API_KEY": "<API_KEY>",
        "TEMPORAM_API_BASE": "https://api.temporam.com"
      }
    }
  }
}
```

## Naming

| Resource | Meaning |
|----------|---------|
| inbound address | An unregistered address generated client-side from a system domain |
| `mailboxes` | Registered outbound sender addresses used by `messages` |
| `emails` | Inbound mail (inbox) |
| `messages` | Outbound send |

## Auth

Get an API key from the Temporam console. Put it in MCP env `TEMPORAM_API_KEY` or the HTTP `Authorization` header. Never commit keys or paste them into skill files.

## Workflow (wait for a verification code)

1. `GET /v3/domains` — pick an available domain.
2. Generate a high-entropy local part client-side (for example, a UUID) and append `@<domain>`. The address does not need to be created or registered.
3. Give the generated address to the third party.
4. Poll `GET /v3/emails/latest?email=` or list with `GET /v3/emails?email=`, then read a result with `GET /v3/emails/{id}`.
5. Read the code from `content`. List items only include `summary`; full body is on detail/latest.

Do **not** call `POST /v3/mailboxes` for inbound mail. Mailbox CRUD only manages outbound sender addresses.

## Quotas

- Claiming one unclaimed inbound message = 1 **inbound** point. `list_emails` may claim multiple messages up to its limit; latest/detail may claim one. Already-owned messages do not charge again.
- With inbound remaining at 0 you can still list your own history; you do not get `429` only because unclaimed mail still exists.
- A successful send = 1 **outbound** point. Hobby plans have `outbound=0` and receive `429` `quota_exceeded`.
- `POST /v3/messages` is **not idempotent** — do not retry blindly.

## Sending (optional)

Create a sender with `POST /v3/mailboxes` if needed. `from` must be one of your active sender mailboxes. Provide `text` and/or `html`. No attachments or CC. `GET /v3/messages/{id}` does not return the body.
