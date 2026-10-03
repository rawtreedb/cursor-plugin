> **This plugin now lives in [rawtreedb/agent-skills](https://github.com/rawtreedb/agent-skills). This repo is archived.**

# RawTree for Cursor

Cursor plugin that connects agents to [RawTree](https://rawtree.com) through RawTree's official hosted [Model Context Protocol](https://modelcontextprotocol.io/) server, and bundles the official RawTree agent skill.

RawTree is a schema-free OLAP database for agents. Send it raw JSON, events, or telemetry; a table is created on the first insert; read the data back with read-only SQL. This plugin lets Cursor explore your organizations, clusters, databases, and tables, ingest JSON events, run SQL, and read RawTree's query and insert logs from chat.

> **RawTree account required.** The plugin only connects Cursor to RawTree; you need your own RawTree account to sign in. RawTree's website currently offers a **Request access** flow ([rawtree.com/waitlist](https://rawtree.com/waitlist)) and RawTree's launch posts describe a private beta, so access may be limited. If you have no account, the sign-in step will not complete. Contact: [contact@rawtree.com](mailto:contact@rawtree.com).

## What's included

| Component | Path | What it is |
| --- | --- | --- |
| MCP server | `mcp.json` | URL-only connector to `https://mcp.rawtree.com/mcp` (Streamable HTTP, OAuth), declared with `"placement": "server"` (hosted MCP path, like other OAuth connectors in `cursor/plugins`). |
| Skill | `skills/rawtree/` | The official RawTree skill: how to use the MCP tools, CLI, and API, write read-only SQL, handle Dynamic (schema-free) fields, and tune performance. |

## Install

1. Open **Cursor Settings → Plugins**.
2. Search for **RawTree**.
3. Click **Install**, then complete the RawTree sign-in prompt.

Or run `/add-plugin rawtree` in chat.

## MCP

```json
{
  "mcpServers": {
    "rawtree": {
      "type": "http",
      "url": "https://mcp.rawtree.com/mcp",
      "placement": "server"
    }
  }
}
```

Auth is OAuth. Your MCP client opens RawTree in a browser, you sign in and approve access, and no API key or header is needed ([RawTree MCP docs](https://rawtree.com/docs/reference/mcp)). RawTree's OAuth server uses dynamic client registration for public clients and Authorization Code with PKCE (`S256`); there is no client secret ([rawtree-platform `docs/auth.md`](https://github.com/rawtreedb/rawtree-platform/blob/main/docs/auth.md)). Each user signs in with their own RawTree account. One connection can reach every organization, cluster, and database the signed-in user can access; resource context is passed per tool call (`organization`, `cluster`, optional `database`) rather than in the MCP URL.

## Security: this connection has broad access

**The OAuth grant gives the agent broad access to your RawTree account, including destructive operations.** RawTree's own docs say: "The OAuth connection grants broad RawTree access, including destructive operations." Tools such as `delete-table`, `delete-database`, `delete-api-key`, `create-api-key`, `remove-organization-member`, `pause-cluster`, and `uninstall-app` are exposed. The RawTree API enforces your role and resource access on every call, but treat the connection as having your full permissions.

- Review every write or admin tool call before approving it; confirm the exact organization, cluster, database, and table. Deletions cannot be undone.
- Be mindful of prompt injection: data you query or ingest and log text returned by `list-logs` are untrusted input to the agent.
- Do not paste API keys into chat. `create-api-key` returns a key that is shown once.
- You can revoke the connection at any time in RawTree under **Profile → OAuth apps** (per RawTree's MCP docs), and remove the plugin in Cursor Settings → Plugins. According to RawTree's auth notes, grant revocation takes effect immediately on the API, even for an unexpired access token.

### What the OAuth grant covers

RawTree currently publishes a single OAuth scope, `full_access` (rawtree-platform `docs/auth.md`). It allows agent-eligible, non-billing operations: organizations, members, clusters, databases, API keys, ingestion, queries, logs, usage, and destructive resource actions. Cluster creation and resizing can incur charges; RawTree's consent screen warns about this. It does **not** allow billing, payment or subscription operations, organization spending-limit changes, RawTree sign-in/session operations, OAuth grant management, or staff admin routes. Access tokens last 15 minutes; refresh tokens rotate and have a sliding 60-day lifetime.

## Tools

The hosted server is the source of truth for tool names and schemas. This list matches the 38 tools registered in [rawtreedb/rawtree-mcp](https://github.com/rawtreedb/rawtree-mcp) at commit `1a37193` (the commit the hosted worker in rawtree-platform pins), and the tools the hosted server listed on 2026-10-02. RawTree's public MCP docs page lists fewer tools, so treat it as a subset.

| Category | Tools |
| --- | --- |
| Discovery | `list-organizations`, `list-clusters`, `get-cluster`, `list-databases`, `list-tables`, `describe-table`, `list-cluster-sizes` |
| Query | `run-query` (read-only SQL; mutating statements are rejected) |
| Ingest | `insert-json` (one object or an array; creates the table on first insert), `insert-from-url` (public JSON/JSONL URL) |
| Logs | `list-logs` (recent insert/select/describe/explain activity with errors and hints) |
| Tables and databases | `create-table`, `update-table` (sorting key), `delete-table`, `create-database`, `delete-database`, `verify-database-s3-access` |
| Clusters | `create-cluster`, `update-cluster`, `pause-cluster`, `resume-cluster`, `verify-cluster-s3-access` |
| Connectors (Kafka) | `list-connectors`, `get-connector`, `get-connector-metrics`, `create-connector`, `add-connector-destination`, `set-connector-status` |
| API keys | `list-api-keys`, `create-api-key`, `delete-api-key` |
| Apps | `list-apps`, `install-app`, `uninstall-app` |
| Organization members | `list-organization-members`, `add-organization-member`, `update-organization-member`, `remove-organization-member` |

### Which tools need organization admin

Per the tool descriptions in `rawtree-mcp` (the API makes the final authorization decision), these state that organization-admin access is required: `create-table`, `update-table`, `create-database`, `verify-database-s3-access`, `verify-cluster-s3-access`, `create-cluster`, `update-cluster`, `pause-cluster`, `resume-cluster`, `install-app`, `uninstall-app`, `create-connector`, `add-connector-destination`, `set-connector-status`, `add-organization-member`, `update-organization-member`, and `remove-organization-member`. `delete-table` is described as requiring an admin key, and RawTree's API rejects it for non-admin credentials.

Read-style tools (for example `list-organizations`, `list-clusters`, `get-cluster`, `list-apps`, `list-organization-members`, `list-connectors`) need a signed-in user with organization membership. The descriptions of the other tools (queries, inserts, logs, `delete-database`, and the API-key tools) do not state an organization-admin requirement for OAuth users; the API-key tool descriptions refer to admin API-key permission. Whatever the tool says, the RawTree API enforces your role. Destructive and provisioning tools tell the agent to confirm the exact target with you before calling them.

## Example prompts

- "List my RawTree organizations, clusters, and databases."
- "Show the tables in my default RawTree database and describe the `events` table."
- "Insert this JSON event into a RawTree table called `app_events`, then describe the table."
- "Query RawTree: daily event counts for the last 7 days from `app_events`, with a LIMIT."
- "Check my RawTree logs for failed inserts in the last hour and explain the errors."
- "Load this public JSONL file into a RawTree table and verify the row count."
- "Review this RawTree SQL query for correctness and performance."

## Notes

- Features, tools, and limits may change; RawTree's access policy is set by RawTree.
- Data you ingest and query is stored in your RawTree cluster. Usage terms and billing are governed by RawTree's own terms.
- For headless agents that cannot open a browser, RawTree's docs describe sending a RawTree API key as a Bearer token to the same hosted endpoint (`Authorization: Bearer rt_...`). Such calls may omit `organization`, `cluster`, and `database` to use the key's bound cluster, and user-level tools such as `list-organizations` require OAuth. This plugin uses OAuth only and does not store or configure keys.

## Links

- Website: https://rawtree.com
- Docs: https://rawtree.com/docs
- MCP reference: https://rawtree.com/docs/reference/mcp
- Authentication: https://rawtree.com/docs/reference/authentication
- Contact: contact@rawtree.com
- Server URL: https://mcp.rawtree.com/mcp
- Official skills repo: https://github.com/rawtreedb/agent-skills

## Attribution and license

This plugin is authored and maintained by gnzjgo ([@gnzjgo](https://github.com/gnzjgo), gonzalo@tinybird.co). The bundled skill in `skills/rawtree/` is copied from [rawtreedb/agent-skills](https://github.com/rawtreedb/agent-skills) (Copyright 2026 RawTree, Apache-2.0), and the hosted MCP server is operated by RawTree. This plugin is licensed under the [Apache License 2.0](LICENSE). The RawTree name and logo are trademarks of their owner; the logo is RawTree's mark from [rawtreedb/rawtree-platform](https://github.com/rawtreedb/rawtree-platform) (`web/public/favicon.svg`).
