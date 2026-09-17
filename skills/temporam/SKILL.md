---
name: temporam
description: Use Temporam temporary email — generate and favorite inbound addresses, wait for messages (OTP/verification), and optionally create sender mailboxes to send mail via API or MCP.
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
| inbound address | An unregistered address generated client-side from a domain in `GET /v3/domains` (system plus the caller's private domains) |
| `mailboxes` | Registered outbound sender addresses used by `messages` |
| `favorites` | Saved inbound addresses for reuse; not sender mailboxes |
| `emails` | Inbound mail (inbox) |
| `messages` | Outbound send |

## Auth

Get an API key from the Temporam console. Put it in MCP env `TEMPORAM_API_KEY` or the HTTP `Authorization` header. Never commit keys or paste them into skill files.

## Workflow (wait for a verification code)

1. `GET /v3/domains` — pick an available domain (`scope=system` public pool, or `scope=user` for a domain this account already owns).
2. Generate a high-entropy local part client-side (for example, a UUID) and append `@<domain>`. The address does not need to be created or registered.
3. Give the generated address to the third party.
4. Poll `GET /v3/emails/latest?email=` or list with `GET /v3/emails?email=`, then read a result with `GET /v3/emails/{id}`.
5. Read the code from `content`. List items only include `summary`; full body is on detail/latest.

Do **not** call `POST /v3/mailboxes` for inbound mail. Mailbox CRUD only manages outbound sender addresses.

## Reusing inbound addresses

Save an inbound address with MCP `create_favorite` or `POST /v3/favorites`. Favorite CRUD does not consume monthly quota and does not create or delete inbound messages.

- Use `list_favorites` to retrieve saved addresses, then pass a favorite's `address` to `list_emails` or `get_latest_email`.
- Use `update_favorite` to replace a saved address and `delete_favorite` to remove it.
- A favorite is not a sender mailbox. To send from that address, pass its `domain` and `local_part` to `create_mailbox`; normal domain availability and mailbox limits still apply.

## Quotas

Call `GET /v3/me` (MCP `get_me`) for name, plan, remaining inbound/outbound, mailbox slots, and period end. It does not consume quota.

- Claiming one unclaimed inbound message = 1 **inbound** point. `list_emails` may claim multiple messages up to its limit; latest/detail may claim one. Already-owned messages do not charge again.
- With inbound remaining at 0 you can still list your own history; you do not get `429` only because unclaimed mail still exists.
- A successful send = 1 **outbound** point. Hobby plans have `outbound=0` and receive `429` `quota_exceeded`.
- Favorite create/list/get/update/delete operations do not consume inbound or outbound quota.
- `POST /v3/messages` is **not idempotent** — do not retry blindly.

## Sending (optional)

Create a sender with `POST /v3/mailboxes` if needed. `from` must be one of your active sender mailboxes. Provide `text` and/or `html`. No attachments or CC. `GET /v3/messages/{id}` does not return the body.

## Contract

Machine-readable spec: import `openapi/openapi.yaml` from the Public API repo (v0.7.0+). Do not copy a second field table into this skill.
