# BUILD REPORT — skillfolio

Clean-room rebuild of `com.emdly/emdly` v1.0.0
(remote-only MCP server at `https://emdly.com/mcp`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/skillfolio` 0.1.0, registry name
`io.github.CloudCorpRecords/skillfolio`.

## Surface built

Labelled subset: **3 of 15 captured tools built**,
12 omitted (see below).

| Tool | Required fields as served |
|------|---------------------------|
| `search_skills` | `query` |
| `get_skill` | `skill` |
| `get_collection` | (none — no `required` key) |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, exactly as captured: the capture's
tools carry `title` keys and they are served verbatim. The tool
literals were generated programmatically from
`harness/fixtures/emdly-tools.json` by the script at
`/tmp/b52-skillfolio-scratch/gen.py` (parse + re-serialise; the
generator's round-trip assertions confirmed the emitted literal
parses back to the fixture's tool objects) — never retyped — and
the registration-integrity test suite plus the parity harness
confirmed the served literals deep-equal the fixture's tool
objects. Tools are registered via the MCP SDK low-level Server
API (`ListToolsRequestSchema` / `CallToolRequestSchema`
handlers), not McpServer+zod.

## Omitted tools (labelled subset)

Absent — asserted in tests at registry level (TOOL_DEFS /
TOOL_NAMES / isKnownTool), MCP level (handleCallTool
MethodNotFound with no network hop, plus a live MCP
tools/list over an in-memory transport serving exactly the
3 fixture tools), and CLI level (each name exits 2 with
usage):

- `install_skill` — returns a third-party skill playbook plus
  a local path and expects the calling agent to write it into
  its own trusted skill directory: an install/write leg into
  the agent's own trusted surface. The plain read of the same
  skill is already served by `get_skill`.
- `submit_skill` — submits a skill for publication under the
  caller's signed-in account: an account-operated submission
  send.
- `brain_list`, `brain_search`, `brain_get`, `brain_write`,
  `brain_create` — the vendor's private per-account memory
  store ("brain"), operated under the caller's sign-in /
  bearer credential: a token-operated private store.
- `github_repos`, `github_read`, `github_search`,
  `github_create_issue`, `github_propose_change` — operate
  the caller's GitHub repos connected through the vendor's
  GitHub App credential: a per-account credential-operated
  surface; create_issue and propose_change are additionally
  writes under that credential.

## Fixture conflicts

**None.** The spec and the fixture agree on the tool names,
titles, descriptions, required fields and property shapes for
all 3 served tools. (For the record: `get_collection`'s
inputSchema in the fixture carries no `required` key at all,
which the spec's "get_collection (no required key)" matches,
and the served literal preserves that.)

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://emdly.com/mcp`.
- Env override `SKILLFOLIO_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization`
  header is sent under any environment — including with key-like
  env vars (`SKILLFOLIO_API_KEY`, `API_KEY`, `AUTHORIZATION`)
  set — and that no `X-API-Key` header is ever sent; upstream
  requests carry only `Content-Type`, `Accept` and, when issued,
  `Mcp-Session-Id`.
- Error rendering `skillfolio error:` text for upstream
  failures — labelled reconstruction.
- Upstream results pass through verbatim, including
  `structuredContent`, `isError` and extra fields. tools/call is
  forwarded as streamable-HTTP JSON-RPC: initialize
  (protocolVersion `2025-03-26`) → notifications/initialized →
  tools/call, honouring `Mcp-Session-Id` when issued; plain-JSON
  and SSE replies are both parsed. The forwarding core takes an
  injectable fetch; tests stub the upstream — no live network
  calls in tests.

## Deviations

**None.**

## Lockfile

- Route: Route 3 (offline) succeeded on the first attempt:
  `npm install --package-lock-only --ignore-scripts --offline`,
  generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile.
- Parses as JSON; `name` == `@reneturcios/skillfolio`.
- Package entries: **128**.
- sha256: `ade4430cc0cbcb328574aed3c57492fd216737f96a47fa8145b09ed0bd767b08`
  (re-verified unchanged after the subsequent full `npm install`).

## server.json

- Description exactly: "Public agent-skills catalogue: search skills, read a skill in full, and browse its collections." —
  **95 characters** (verified programmatically from the file).
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/skillfolio:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/skillfolio"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/skillfolio`
  and its smoke test asserts the server name `skillfolio`.

## Verification (all run 2026-10-28)

1. `tsc --noEmit` (direct binary) → exit 0.
2. `tsc` build → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. Tests (`tsx --test test/forward.test.ts`, fake-fetch only) →
   **62 tests, 62 pass, 0 fail** (registration-integrity suite
   deep-equals the served tools/list against the fixture's 3
   tools, including title/description verbatim checks,
   required-key exactness including get_collection's absent
   required key, and property-shape checks; omitted-tool
   absence asserted at registry, MCP (handler + in-memory
   tools/list + tools/call) and CLI levels for all 12 omitted
   tools; plus session handling/reuse, env override, the
   no-Authorization / no-X-API-Key assertions under any
   environment, verbatim argument forwarding for all 3 tools
   and result pass-through including structuredContent and
   upstream isError, SSE parsing, error rendering for both the
   JSON-RPC-error and transport-failure paths, MCP-layer
   validation, and CLI behaviour including a working subcommand
   for each of the 3 tools against a local stub server).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/emdly-tools.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/skillfolio/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/skillfolio/parity-out` → **verdict PASS, parity 100.0**
   (3/3 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
