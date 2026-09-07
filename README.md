# Mailercloud plugin for Claude Code

Bundles the Mailercloud MCP server together with ready-made commands, a best-practices skill,
and a campaign-manager subagent — so Claude Code users get expert Mailercloud workflows out of
the box, not just the raw tools.

> This is a **Claude Code** plugin. For claude.ai / Desktop / Mobile, use the Mailercloud
> **connector** in the Claude directory instead — same server, broader reach.

## What's inside
| Piece | What it adds |
|-------|--------------|
| `.mcp.json` | Connects the Mailercloud MCP server (`https://mcp.mailercloud.com/mcp`, 47 tools) |
| `/mailercloud:campaign-audit` | Audit a draft campaign before sending |
| `/mailercloud:analyze-campaigns` | Analyze recent campaigns + action plan |
| `mailercloud-best-practices` skill | Tool map, pre-send checklist, deliverability & metric guidance |
| `campaign-manager` agent | A specialist persona for multi-step campaign work |

No changes to the Mailercloud server are required — the plugin only references it.

## Requirements
- A Mailercloud account. On first tool use, Claude Code runs the OAuth flow (or accepts a
  Mailercloud API key) to authorize.

## Try it locally
From this repo's parent directory, add it as a local marketplace and install:

```
/plugin marketplace add ./mailercloud-plugin
/plugin install mailercloud@mailercloud
```

Then run `/mailercloud:campaign-audit` or ask the campaign-manager agent to review your sends.

## Publish
1. Push this folder to a public GitHub repo (e.g. `mailercloud-com/mailercloud-plugin`).
2. Users install with:
   ```
   /plugin marketplace add mailercloud-com/mailercloud-plugin
   /plugin install mailercloud@mailercloud
   ```

## Structure
```
mailercloud-plugin/
  .claude-plugin/
    plugin.json          # plugin manifest
    marketplace.json     # makes this repo installable as a marketplace
  .mcp.json              # references the remote Mailercloud MCP server
  commands/              # /mailercloud:* slash commands
  skills/                # mailercloud-best-practices
  agents/                # campaign-manager
```
