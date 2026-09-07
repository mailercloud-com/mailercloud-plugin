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

## Install

In Claude Code, run:

```
/plugin marketplace add https://github.com/mailercloud-com/mailercloud-plugin.git
/plugin install mailercloud@mailercloud
```

> **Use the full `https://….git` URL above.** The GitHub shorthand
> (`mailercloud-com/mailercloud-plugin`) makes Claude Code clone over SSH, which fails if you
> don't have GitHub SSH keys configured. The HTTPS URL works for everyone.

Then restart Claude Code if prompted, and try:

```
/mailercloud:campaign-audit
```

## Requirements & authentication
- A **Mailercloud account** (sign up at https://mailercloud.com).
- The first time a Mailercloud tool runs, Claude Code connects the MCP server and prompts you
  to **authorize** — sign in to Mailercloud (OAuth) or provide a Mailercloud API key. Until you
  authorize, the tools will report that authentication is required; this is expected.

## Verify it installed
```
/plugin list                     # should show: mailercloud@mailercloud
```
You should also see `/mailercloud:campaign-audit` and `/mailercloud:analyze-campaigns` in the
slash-command list, and the `mailercloud` MCP server under your connected tools.

## Local development
To test changes without publishing, add the folder as a local marketplace:
```
/plugin marketplace add ./mailercloud-plugin
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
