# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted emdly skills-catalogue MCP server. The 3 captured served tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `search_skills`, `get_skill`, `get_collection`. Each tool's `title` is served exactly as captured. This is a labelled subset (3 of 15 captured tools); `install_skill`, `submit_skill`, the five `brain_*` tools and the five `github_*` tools are omitted.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `skillfolio error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `SKILLFOLIO_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
