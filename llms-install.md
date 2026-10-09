# Installing the Mailercloud MCP server (for AI agents)

Mailercloud is a hosted, remote MCP server. There is nothing to install, build or run locally.

- Server URL: `https://mcp.mailercloud.com/mcp`
- Transport: Streamable HTTP
- Authentication: OAuth 2.1 (PKCE, dynamic client registration). The client opens a browser sign-in to Mailercloud the first time a tool is used. Do not ask the user for an API key unless the client cannot do OAuth.
- Requirement: a Mailercloud account (free plan works). Sign up at https://www.mailercloud.com

## Cline

Add this entry under `"mcpServers"` in `cline_mcp_settings.json`
(Cline panel > menu > MCP Servers > Configure):

```json
"mailercloud": {
  "type": "streamableHttp",
  "url": "https://mcp.mailercloud.com/mcp",
  "disabled": false,
  "autoApprove": []
}
```

Save the file. When the first Mailercloud tool runs, ask the user to complete the Mailercloud sign-in in the browser.

## Fallback: API key

Only if the client cannot complete OAuth: ask the user for their Mailercloud API key
(Mailercloud > Settings > API) and add a header to the same entry:

```json
"headers": { "Authorization": "Bearer <MAILERCLOUD_API_KEY>" }
```

Never print or commit the key.

## Other MCP clients

Any client that supports remote MCP servers can use the same URL:

- Claude Code: `claude mcp add --transport http mailercloud https://mcp.mailercloud.com/mcp`
- Gemini CLI: `gemini extensions install https://github.com/mailercloud-com/mailercloud-plugin`
- VS Code, Cursor, Windsurf and others: add an HTTP server with URL `https://mcp.mailercloud.com/mcp`

## Verify

Call the `list_contact_lists` tool. If it returns the user's contact lists, setup worked.

## Safe use

- Before `schedule_campaign` or `send_transactional_email`, show the campaign name, list, sender and send time, and ask the user to confirm.
- Never delete contacts, lists or webhooks without explicit confirmation.
