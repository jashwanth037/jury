# @jury/mcp-server

Model Context Protocol (MCP) server for **JURY**, enabling AI coding assistants (Claude Desktop, Antigravity, Cursor, Cline) to directly interact with, audit, and simulate payment gateway operations.

---

## Capabilities & Tools

| Tool Name | Description |
| :--- | :--- |
| **`JURY_check_gateway_health`** | Inspects HTTP operational readiness across ports (6100 Core, 7000 Checkout, 6001 Admin, 6002 SMS, 6003 Gmail). |
| **`JURY_verify_webhook`** | Verifies HMAC-SHA256 signature over raw request bytes and checks timestamp freshness. |
| **`JURY_simulate_bank_notification`** | Formats realistic Indian bank SMS templates (SBI, HDFC, ICICI, Axis, Kotak, PNB) and generates mock observer payloads. |

---

## Configuration

Add to your MCP client configuration (`claude_desktop_config.json` or Antigravity `mcp_config.json`):

```json
{
  "mcpServers": {
    "jury": {
      "command": "node",
      "args": ["packages/jury-mcp/src/index.js"],
      "env": {
        "NODE_ENV": "development"
      }
    }
  }
}
```

---

## Running Standalone

```bash
node packages/jury-mcp/src/index.js
```
