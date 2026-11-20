# VS Code

Server URL: `https://app.northboundintelligence.com/api/mcp`
Auth: `Authorization: Bearer <your API key>`

Create an API key in the app (Settings > API Credentials). VS Code's MCP configuration uses
`servers` as its root key (not `mcpServers`) and can prompt for the key so it is never written
to the file.

Add this to `.vscode/mcp.json` in your workspace (or to your user-level MCP configuration):

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "northbound-api-key",
      "description": "NorthBound API key (nbk_live_...)",
      "password": true
    }
  ],
  "servers": {
    "northbound-sentinel": {
      "type": "http",
      "url": "https://app.northboundintelligence.com/api/mcp",
      "headers": {
        "Authorization": "Bearer ${input:northbound-api-key}"
      }
    }
  }
}
```

VS Code prompts for the key when the server starts, so it does not need to be written into the
file. The word `Bearer` followed by a space is part of the header value; the prompt takes only
the key.

If you prefer an environment variable, VS Code's editor-managed syntax for it is
`${env:NORTHBOUND_API_KEY}` in place of `${input:northbound-api-key}`.

## Verify

Open the Command Palette and run **MCP: List Servers**, then confirm `northbound-sentinel` is
running and its tools appear in Chat (agent mode).
