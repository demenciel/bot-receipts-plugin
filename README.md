# Bot Receipts plugin

Connects your agent to the Bot Receipts remote MCP server (`https://botsreceipt.app/mcp`, Streamable HTTP). It also adds a `log-receipts` skill that tells the agent when and how to report work.

**Auth:** OAuth 2.1 with PKCE and dynamic client registration, handled by the host. The plugin stores no secrets. The first time you use it, sign in at botsreceipt.app and approve access to your workspace.

**Tools:** connection_status, register_bot, start_run, report_progress, submit_receipt, list_pending_feedback, get_feedback, acknowledge_feedback, get_work_summary.

Author: Alex Couture. Homepage: https://botsreceipt.app
