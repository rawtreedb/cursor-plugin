# Changelog

All notable changes to this plugin will be documented here.

## 1.0.0 — initial release

- Added the `rawtree` MCP server pointing at RawTree's hosted Streamable HTTP endpoint (`https://mcp.rawtree.com/mcp`). Auth is OAuth 2.0 with PKCE (browser sign-in); there is no API key, client ID, or header to configure.
- Declared `"placement": "server"` on the MCP server, the same way other URL-only OAuth connectors in `cursor/plugins` (for example `posthog-mcp`, `x-money`, `trello`) do, so Grok Bot dials it through the hosted MCP path. Declared `minClientVersions.cursor` `3.13.0`, the value PostHog and most other hosted-MCP plugins in `cursor/plugins` use.
- Bundled the official `rawtree` skill (`skills/rawtree/`) from [rawtreedb/agent-skills](https://github.com/rawtreedb/agent-skills) (Apache-2.0), skill version 0.5.0 at commit `3b13c3a`. The OpenAI-specific `agents/openai.yaml` was not copied.
- Logo: RawTree's official gradient mark (`web/public/favicon.svg` in rawtreedb/rawtree-platform), rendered at 1024×1024 on an opaque white square with the same proportions as RawTree's `apple-touch-icon.png`.
- README: verified against rawtreedb/rawtree-platform and rawtreedb/rawtree-mcp (revocation under Profile → OAuth apps, `full_access` scope, admin-only tools, Bearer API-key option for headless clients, access wording).
