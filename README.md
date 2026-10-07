# Bot Receipts plugin

Connects your agent to the Bot Receipts remote MCP server (`https://botsreceipt.app/mcp`, Streamable HTTP). It also adds a `log-receipts` skill that tells the agent when and how to report work: start a run, report real milestones or blockers, and submit an honest receipt for owner review.

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
connection_status, register_bot, start_run, report_progress, submit_receipt, list_pending_feedback, get_feedback, acknowledge_feedback, get_work_summary.

## Legal
- Privacy: https://botsreceipt.app/privacy
- Terms: https://botsreceipt.app/terms

Author: Alex Couture · License: MIT · Homepage: https://botsreceipt.app
