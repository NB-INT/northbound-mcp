# Cursor

Server URL: `https://app.northboundintelligence.com/api/mcp`
Auth: `Authorization: Bearer <your API key>`

Create an API key in the app (Settings > API Credentials) and set it as an environment
variable that the Cursor process can see (for example, export it in your shell profile and
start Cursor from that shell, or set it at the OS level):

```bash
export NORTHBOUND_API_KEY="nbk_live_..."
```

Add the server to `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (this project
only). Cursor resolves `${env:NAME}` in `url` and `headers`, and a remote server is
configured with just a `url`:

```json
{
  "mcpServers": {
    "northbound-sentinel": {
      "url": "https://app.northboundintelligence.com/api/mcp",
      "headers": {
        "Authorization": "Bearer ${env:NORTHBOUND_API_KEY}"
      }
    }
  }
}
```

The word `Bearer` plus a space is part of the header value; the environment variable
holds only the key.

## Verify

Open Cursor's settings, go to the MCP section, and confirm `northbound-sentinel` shows as
connected with its tools listed. Restart Cursor after changing the environment variable.
