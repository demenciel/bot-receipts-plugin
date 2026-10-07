# Bot Receipts plugin

Connects your agent to the Bot Receipts remote MCP server (`https://botsreceipt.app/mcp`, Streamable HTTP). It also adds a `log-receipts` skill that tells the agent when and how to report work: on connect, check webhook state and register a wake URL when the runtime has one; then check pending feedback, start a run, report real milestones or blockers, submit an honest receipt for owner review, then check feedback again.

Bots must call `connection_status` on connect with `bot_key` (their `external_key`) and `bot_label`. If `webhook.configured` is false and the runtime offers a webhook trigger (for example a Grok Bot webhook routine URL plus an `Authorization` header), they call `register_webhook` with the same `bot_key` and `bot_label`, `url`, and `auth_header_value` so owner reviews wake this bot. Omitting `bot_key` attaches the webhook to the wrong bot. Never print the signing secret or `auth_header_value`.

Bots must still call `list_pending_feedback` before `start_run` and after `submit_receipt`, then read and `acknowledge_feedback` for **every** pending item (including accepted or archived reviews that have notes). Revise the work and submit a new receipt only for `changes_requested`, then acknowledge.

## Install

### From the marketplace (once listed)
Open the Cursor or Grok Bot plugin marketplace, search for **Bot Receipts**, and install it. The host then starts the sign-in flow below the first time a tool is used.

### Manual MCP config
If you just want the MCP server, add this to your MCP config (for example `~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "bot-receipts": {
      "url": "https://botsreceipt.app/mcp"
    }
  }
}
```

There are no API keys to paste. Auth is OAuth 2.1 with PKCE and dynamic client registration, and your host handles it.

## Signing in
1. Your host opens botsreceipt.app. Sign in with Google or GitHub.
2. A **Connect to Bot Receipts** screen asks whether to allow your client (shown by name) to report work in your workspace. It shows where access returns to, warns you when the client is a local app, and lists the requested scopes as checkboxes.
3. Click **Approve connection**, or **Cancel** to deny.

Approving grants reporting and feedback access. It does not grant billing, owner acceptance, deletion, or account administration.

### Scopes
- `reports:read`: read feedback and your work summary.
- `reports:write`: register bots, start runs, report progress, submit receipts, acknowledge feedback, and register or remove a wake webhook.

Keep both scopes checked, or the reporting tools won't work.

> **Connected before the write scope was added?** Older connections may only hold `reports:read`. If tool calls fail with an authorization error, disconnect and sign in again so the host requests both scopes.

## Tools
- `connection_status` — on connect; pass `bot_key` (this bot's `external_key`) and `bot_label`. Returns masked `webhook`: `configured`, `scope`, `url`, `secret_hint`, `auth_header_name`, `auth_header_hint`
- `register_webhook` — same `bot_key`/`bot_label`, `url`, optional `auth_header_name` (default `Authorization`), `auth_header_value`, `clear_auth_header`. Returns the signing secret once (never print the secret or `auth_header_value`). Re-registering without `auth_header_value` keeps the existing header unless `clear_auth_header` is `true` (being added in botsreceipt #26)
- `remove_webhook` — same `bot_key`/`bot_label`; remove this bot's webhook
- `register_bot` — stable bot label; reuse the returned bot id. `bot_key` is this `external_key`
- `start_run` — begin a run (after pending feedback is handled)
- `report_progress` — real milestones or blockers (`running` | `blocked` | `awaiting_owner`)
- `submit_receipt` — end of work (`complete` | `partial` | `failed` | `blocked`)
- `list_pending_feedback` — optional `run_id`, `cursor`; up to 50 unacknowledged items with full notes; call before every `start_run` and after every `submit_receipt`
- `get_feedback` — optional `run_id`, `cursor`; up to 50 items of all feedback, acknowledged or not
- `acknowledge_feedback` — after each pending item is handled (not owner acceptance)
- `get_work_summary` — read-only overview

## Owner webhook
On connect, call `connection_status` with `bot_key` and `bot_label`. If `webhook.configured` is false and the runtime has a trigger (Grok Bot: routine URL + `Authorization` header), call `register_webhook` with the same `bot_key`/`bot_label`, `url`, and `auth_header_value` so reviews wake this bot. Owners can also set a URL in the dashboard. Never print the signing secret or `auth_header_value`.

The service POSTs JSON on `receipt.reviewed`, `receipt.submitted`, and `webhook.test`. HTTPS only; no redirects. There is no `type`, `id`, `workspace_id`, or `review_id` field; the delivery id is only `X-BotReceipts-Delivery`. Treat the POST as a hint and call `list_pending_feedback`. A webhook does not replace pull-based checks.

**Headers**
- `X-BotReceipts-Signature: t=<ts>,v1=<hex>` — `v1` is the hex HMAC-SHA256 of `` `${ts}.${rawBody}` `` using the endpoint secret (`brwhsec_` prefix)
- `X-BotReceipts-Timestamp` — same unix-seconds `t`
- `X-BotReceipts-Delivery` — delivery id for dedupe
- Optional caller-supplied auth header (`auth_header_name` / `auth_header_value`)

Reject when `|now - t| > 300` (either direction).

**Retries:** 5xx, 408, 429, and network errors, with backoff from 30s up to 6h over 8 attempts. No retry on other 4xx. `webhook.test` is sent once.

**Payloads**
- `receipt.reviewed`: `{event, feedback_id, receipt_id (or null), run_id, bot_id, bot_external_key, action ('accepted'|'changes_requested'), note (always present, may be ''), created_at, dashboard_url}`
- `receipt.submitted`: `{event, receipt_id, run_id, bot_id, bot_external_key, created_at, dashboard_url}`
- `webhook.test`: `{event, created_at, dashboard_url}`

A Grok Bot routine with a webhook trigger *can* wake the bot. The routine may not verify HMAC. Prefer a small relay that verifies the signature when available, or use the event only as a hint to call `list_pending_feedback`.

## Legal
- Privacy: https://botsreceipt.app/privacy
- Terms: https://botsreceipt.app/terms

Author: Alex Couture · License: MIT · Homepage: https://botsreceipt.app
