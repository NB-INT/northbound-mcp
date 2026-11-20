# NorthBound Sentinel MCP server

Connect your AI coding agent to [NorthBound Sentinel](https://northboundintelligence.com)
over the [Model Context Protocol](https://modelcontextprotocol.io). Your agent can see which
of your domains are exposed, read your findings with their evidence and fix guidance, pull a
signed report, and start a governed assessment, without leaving the editor.

This repository holds documentation and client configuration only. The server is hosted by
NorthBound; there is nothing to install or run locally.

## Who it is for

Developers and "vibe-coders" who build with AI agents (Claude Code, Cursor, VS Code, and other
MCP clients) and want the same agent that writes the code to check how exposed the result is
on the internet, and to help fix what it finds.

## Server details

| Item | Value |
| --- | --- |
| Server URL | `https://app.northboundintelligence.com/api/mcp` |
| Transport | Streamable HTTP (JSON-RPC 2.0 over `POST`) |
| Authentication | API key in the `Authorization` header: `Authorization: Bearer <your API key>` |
| MCP protocol versions | `2025-03-26`, `2024-11-05` |

Available on every plan, including the free plan. Usage is metered: each plan includes a
monthly allowance, and tool calls count against your account's API usage.

## 1. Get an API key

1. Sign in to the app at <https://app.northboundintelligence.com>.
2. Open **Settings > API Credentials** and create a new credential.
3. Choose only the scopes you need (see below) and, optionally, restrict the key to an IP
   allowlist.
4. Copy the key right away. It starts with `nbk_live_` and is shown only once. If you lose it,
   rotate it or create a new one.

Treat the key like a password: keep it in an environment variable or secret store, never in a
repository.

| Scope | What it lets the agent do |
| --- | --- |
| `read_assets` | Read your projects, assets, discovered surface and assessment runs |
| `read_findings` | Read findings, your posture summary and generate reports |
| `launch_scans` | Start assessments (leave this off if you only want read access) |

## 2. Connect your client

Put the key in an environment variable named `NORTHBOUND_API_KEY`, then add the server to your
client. The header value is the word `Bearer`, a space, then the key.

- Claude Code: [snippets/claude-code.md](snippets/claude-code.md)
- Cursor: [snippets/cursor.md](snippets/cursor.md)
- VS Code: [snippets/vscode.md](snippets/vscode.md)
- Any other MCP client that supports remote HTTP servers with custom headers: use the server
  URL and the header above, or copy [`.mcp.json`](.mcp.json).

To check that your key works before configuring a client:

```bash
curl -sS https://app.northboundintelligence.com/api/mcp \
  -H "Authorization: Bearer $NORTHBOUND_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

A valid key returns the list of available tools. `401` means the key was not accepted (see
Troubleshooting).

## What your agent can do

Exact tool names and parameters are published by the server itself (`tools/list`), so your
client always shows the current set. In plain terms:

- **See your inventory.** List your projects and domains, with each domain's ownership
  verification status and open-finding counts.
- **See your exposed surface.** For a domain, retrieve the application map the platform
  discovered, its endpoints, and the technologies it runs together with known-vulnerability
  alerts.
- **Read findings.** Search and filter findings by project, domain, severity, status or
  keyword, and open any finding to get its evidence, proof level, severity scoring and
  remediation guidance.
- **Get a posture summary.** Open findings by severity, how much of your inventory is
  ownership-verified, remediation deadlines that are slipping, and the top things to fix next.
- **Follow assessments.** List past and running assessments and poll the status of one.
- **Generate reports.** Produce a report for a project or a single domain as a structured
  summary, a standalone HTML document, or a digitally signed PDF.
- **Launch an assessment** on a domain you own, with the safeguards below.

Example prompts:

- "List my NorthBound assets and tell me which ones have open high or critical findings."
- "Show me the evidence and the fix guidance for that finding, then propose a patch."
- "Start a passive assessment on staging.example.com and tell me when it finishes."
- "Generate the signed report for this project."

## Safeguards built into the server

- **Your data only.** Every call is limited to your own account. Results never include another
  customer's data.
- **Least privilege.** Keys carry scopes; a read-only key cannot start an assessment.
- **Ownership first.** Assessments that interact with a target more actively than passive
  observation (the Balanced and Active postures) require that you have verified ownership of
  the domain. Unverified domains accept Passive assessments only.
- **A human confirms authorization.** Launching an assessment requires an explicit
  `legal_authorization: true`, which records in the audit trail that you are legally
  authorized to test the target. Your agent must ask you; it must not set this on its own.
- **One at a time, rate limited.** Assessments run sequentially per user, one launch per
  request, and calls are rate limited per key.
- **Client-safe output.** Results use the same client-facing wording as the NorthBound app and
  reports.

## Troubleshooting

| Symptom | Likely cause |
| --- | --- |
| `401` | Key missing, mistyped, expired or revoked; the request comes from an IP outside the key's allowlist; or the header is not `Authorization: Bearer <key>` |
| `402` | Your current plan or usage allowance does not cover the call; check your plan in the app |
| Tool result "insufficient scope" | The key lacks the scope that tool needs; create or edit a key with it |
| `429` or "rate limited" | Too many calls or launches in a short time; wait and retry |
| Variable shows up literally as `${NORTHBOUND_API_KEY}` | The environment variable is not set in the process that launched your client |

## About NorthBound and what this is not

NorthBound is a human-led, AI-assisted security team. This server gives your agent governed
access to your own results and to assessments you are authorized to run. It supports your
review; it does not replace it, and it is not a compliance certification or a guarantee that a
system is secure.

To report a vulnerability in NorthBound itself, see <https://northboundintelligence.com/security/>.

Website: <https://northboundintelligence.com>

Documentation and configuration in this repository are released under the [MIT License](LICENSE). The NorthBound service itself is proprietary.
