# skillfolio

Looking for a ready-made playbook before starting a task from scratch? skillfolio is a local forwarding client for the hosted emdly skills-catalogue MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

The three tools read emdly's published skills catalogue. Nothing here installs a skill, submits anything, opens an account, or touches a private store — every tool is a public catalogue read.

## The tools

- **`search_skills`** — Search the published catalogue for a ready-made skill. Pass 1–3 short keywords as `query` (required); words are matched independently, so a sentence works too. An empty query returns the most popular skills. An optional `category` narrows the search. Results are ranked skills with their owner/name id, category, description and install command.
- **`get_skill`** — Read one published skill's Markdown in full, to inspect it without installing anything. `skill` is required, in owner/name form (for example `opsmith/jira-ticket-scorer`); an optional integer `version` pins a specific version instead of the latest.
- **`get_collection`** — Browse curated bundles of skills. With no argument, it lists the available collections (slug, name, size). With a `collection` slug, it returns the skills in that collection, in order.

This is a labelled subset of the upstream surface: 3 of the 15 captured tools are served. The other 12 — the install and submission tools, the per-account "brain" memory-store tools, and the GitHub tools that operate a caller's connected repositories — are omitted and are not present in this rebuild at any level.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/skillfolio mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "skillfolio": {
      "command": "npx",
      "args": ["-y", "@reneturcios/skillfolio", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/skillfolio
docker run --rm ghcr.io/cloudcorprecords/skillfolio search_skills '{"query":"code review"}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
skillfolio search_skills '{"query":"code review"}'
skillfolio search_skills '{"query":"","category":"code-review"}'
skillfolio get_skill '{"skill":"opsmith/jira-ticket-scorer"}'
skillfolio get_collection '{}'
skillfolio get_collection '{"collection":"laravel-starter"}'
skillfolio --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `SKILLFOLIO_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://emdly.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `skillfolio error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (the 3 tools, deep-equal, titles included exactly as captured, and each `required` list exactly as captured), asserts that all 12 omitted tools are absent at registry, MCP and CLI level, exercises the session handshake and its reuse, verifies that no `Authorization` or `X-API-Key` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
