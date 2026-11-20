# Claude Code

Server URL: `https://app.northboundintelligence.com/api/mcp`
Auth: `Authorization: Bearer <your API key>`

First, create an API key in the app (Settings > API Credentials) and export it in the shell
that starts Claude Code:

```bash
export NORTHBOUND_API_KEY="nbk_live_..."
```

## Option A: one command

```bash
claude mcp add --transport http northbound-sentinel \
  https://app.northboundintelligence.com/api/mcp \
  --header "Authorization: Bearer $NORTHBOUND_API_KEY"
```

This stores the expanded key in your local Claude Code configuration, so the key is on disk in
that file. Use Option B if you want the config to hold only a reference.

## Option B: project `.mcp.json` (safe to commit)

Create `.mcp.json` at the project root. Claude Code expands `${VAR}` in `url` and `headers`
when it loads the file, so the key itself never appears in the file:

```json
{
  "mcpServers": {
    "northbound-sentinel": {
      "type": "http",
      "url": "https://app.northboundintelligence.com/api/mcp",
      "headers": {
        "Authorization": "Bearer ${NORTHBOUND_API_KEY}"
      }
    }
  }
}
```

Note that the word `Bearer` plus a space is part of the header value; the environment
variable holds only the key.

## Verify

Run `claude mcp list` (or `/mcp` inside Claude Code) and check that `northbound-sentinel` is
connected. If the variable is not set, Claude Code warns and sends the literal `${...}` text,
which the server rejects with `401`.
