# Bot Receipts plugin

Connects your agent to the Bot Receipts remote MCP server (`https://botsreceipt.app/mcp`, Streamable HTTP). It also adds a `log-receipts` skill that tells the agent when and how to report work: check pending feedback, start a run, report real milestones or blockers, submit an honest receipt for owner review, then check feedback again.

Bots must call `list_pending_feedback` before `start_run` and after `submit_receipt`. On `changes_requested`, they revise the work, submit a new receipt, and call `acknowledge_feedback` once that item is handled.

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
- `list_pending_feedback` — before every `start_run` and after every `submit_receipt`
- `get_feedback` — one pending item
- `acknowledge_feedback` — after the item is handled (not owner acceptance)
- `get_work_summary` — read-only overview

## Owner webhook
Register a webhook in Bot Receipts to POST on `receipt.reviewed`, `receipt.submitted`, and `webhook.test`.

- Header: `X-BotReceipts-Signature: t=<ts>,v1=<hex>`
- `v1` is the hex HMAC-SHA256 of `` `${ts}.${rawBody}` ``
- Reject timestamps older than 5 minutes

Point the URL at a Grok Bot routine with a webhook trigger to wake the bot when a receipt is reviewed or submitted.

## Legal
- Privacy: https://botsreceipt.app/privacy
- Terms: https://botsreceipt.app/terms

Author: Alex Couture · License: MIT · Homepage: https://botsreceipt.app
