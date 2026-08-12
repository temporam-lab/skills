---
name: temporam
description: Use Temporam temporary email — create mailboxes, wait for inbound messages (OTP/verification), and send mail via API or MCP.
---

# Temporam

Temporary email API. Auth: `Authorization: Bearer <API_KEY>`. Default base URL: `https://api.temporam.cn`.

Follow the public OpenAPI spec. Do not invent endpoints that are not listed there.

## Naming

| Resource | Meaning |
|----------|---------|
| `mailboxes` | Mailbox addresses |
| `emails` | Inbound mail (inbox) |
| `messages` | Outbound send |

## Auth

Get an API key from the Temporam console. Put it in MCP env `TEMPORAM_API_KEY` or the HTTP `Authorization` header. Never commit keys or paste them into skill files.

## Workflow (wait for a verification code)

1. `GET /v3/domains` — pick an available domain.
2. `POST /v3/mailboxes` — create an address (`local_part` optional).
3. Give that address to the third party.
4. Poll `GET /v3/emails/latest?email=` or list then `GET /v3/emails/{id}`.
5. Read the code from `content`. List items only include `summary`; full body is on detail/latest.

## Quotas

- Claiming one unclaimed inbound message = 1 **inbound** point (on latest/detail when the message was unclaimed; already-owned messages do not charge again).
- With inbound remaining at 0 you can still list your own history; you do not get `429` only because unclaimed mail still exists.
- A successful send = 1 **outbound** point. Hobby plans have `outbound=0` and receive `429` `quota_exceeded`.
- `POST /v3/messages` is **not idempotent** — do not retry blindly.

## Sending (optional)

`from` must be one of your active mailboxes. Provide `text` and/or `html`. No attachments or CC. `GET /v3/messages/{id}` does not return the body.

## MCP

If `@temporam/mcp` is installed, prefer its tools (`create_mailbox`, `get_latest_email`, etc.) over raw curl unless the user asks for HTTP.
