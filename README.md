# Bot Receipts plugin

Connects your agent to the Bot Receipts remote MCP server (`https://botsreceipt.app/mcp`, Streamable HTTP). It also adds a `log-receipts` skill that tells the agent when and how to report work: check pending feedback, start a run, report real milestones or blockers, submit an honest receipt for owner review, then check feedback again.

Bots must call `list_pending_feedback` before `start_run` and after `submit_receipt`, then read and `acknowledge_feedback` for **every** pending item (including accepted or archived reviews that have notes). Revise the work and submit a new receipt only for `changes_requested`, then acknowledge.

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
- `reports:write`: register bots, start runs, report progress, submit receipts, and acknowledge feedback.

Keep both scopes checked, or the reporting tools won't work.

> **Connected before the write scope was added?** Older connections may only hold `reports:read`. If tool calls fail with an authorization error, disconnect and sign in again so the host requests both scopes.

## Tools
- `connection_status` — first use only; confirms access
- `register_bot` — stable bot label; reuse the returned bot id
- `start_run` — begin a run (after pending feedback is handled)
- `report_progress` — real milestones or blockers (`running` | `blocked` | `awaiting_owner`)
- `submit_receipt` — end of work (`complete` | `partial` | `failed` | `blocked`)
- `list_pending_feedback` — optional `run_id`, `cursor`; up to 50 unacknowledged items with full notes; call before every `start_run` and after every `submit_receipt`
- `get_feedback` — optional `run_id`, `cursor`; up to 50 items of all feedback, acknowledged or not
- `acknowledge_feedback` — after each pending item is handled (not owner acceptance)
- `get_work_summary` — read-only overview

## Owner webhook
Register an HTTPS endpoint in Bot Receipts. The service POSTs JSON on `receipt.reviewed`, `receipt.submitted`, and `webhook.test`. HTTPS only; no redirects. At-least-once delivery with retries — dedupe.

**Headers**
- `X-BotReceipts-Signature: t=<ts>,v1=<hex>` — `v1` is the hex HMAC-SHA256 of `` `${ts}.${rawBody}` `` using the endpoint secret (`brwhsec_` prefix)
- `X-BotReceipts-Timestamp` — same unix-seconds `t`
- `X-BotReceipts-Delivery` — delivery id for dedupe (being added in botsreceipt #25)

Reject when `|now - t| > 300` (either direction).

**Payload fields:** `id`, `type` (`receipt.reviewed` | `receipt.submitted` | `webhook.test`), `created_at`, `workspace_id`, `bot_id`, `run_id`, `receipt_id`; `receipt.reviewed` also includes `review_id`, `action`, and `note` when present. Treat the POST as a hint and call `list_pending_feedback`.

A Grok Bot routine with a webhook trigger *can* wake the bot, but the routine may not verify HMAC. Prefer a small relay that verifies the signature, or use the event only as a hint to call `list_pending_feedback`.

## Legal
- Privacy: https://botsreceipt.app/privacy
- Terms: https://botsreceipt.app/terms

Author: Alex Couture · License: MIT · Homepage: https://botsreceipt.app
