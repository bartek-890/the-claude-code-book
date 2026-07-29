# MCP

_III · Harness · Chapter 15_

The Model Context Protocol connects Claude Code to external tools and data — issue trackers, databases, design files, monitoring. Connect a server when you find yourself _copying data into chat from another tool_. That is the whole trigger. Everything else is configuration.

## Adding servers

```
# remote HTTP (most common for hosted services)
claude mcp add --transport http notion https://mcp.notion.com/mcp

# with a header
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer TOKEN"

# remote SSE
claude mcp add --transport sse asana https://mcp.asana.com/sse

# local stdio — note the `--` separator
claude mcp add --env AIRTABLE_API_KEY=KEY --transport stdio airtable \
  -- npx -y airtable-mcp-server
```

In JSON configuration the `type` field accepts `streamable-http` as an alias for `http`, so definitions copied from a server’s own docs work unmodified. WebSocket (`type: "ws"`) exists for servers that push events unprompted, but supports neither OAuth nor the `--transport` flag — header auth only.

## Scopes and precedence

| Scope | Loads in | Shared | Stored in |
| --- | --- | --- | --- |
| **Local** (default) | Current project only | No | `~/.claude.json`, under the project entry |
| **Project** | Current project only | Yes, via git | `.mcp.json` in the project root |
| **User** | All your projects | No | `~/.claude.json` |

```
{
  "mcpServers": {
    "shared-server": {
      "command": "/path/to/server",
      "args": [],
      "env": {}
    }
  }
}
```

Project-scoped servers from `.mcp.json` require approval before use — a checked-in file is code someone else can add. `claude mcp reset-project-choices` clears those decisions. Some names are reserved for built-ins and will be skipped with a warning: `workspace`, `claude-in-chrome`, `computer-use`, `Claude Preview`, `Claude Browser`.

## Using what a server provides

- **TOOLS** — Named mcp__<server>__<tool> — the string for permission rules and hook matchers
- **RESOURCES** — Reference with @server:resource in a prompt
- **PROMPTS** — Exposed as slash commands: /mcp__server__prompt
- **CHANNELS** — Push events into a live session — see below

`/mcp` manages connections and runs OAuth flows; `claude mcp login <name>` does it from the command line. Long tool calls are backgrounded automatically rather than blocking the session.

## The cost, and how to keep it down

Every connected server costs tool names at startup. Schemas are deferred by default and load through `ToolSearch` on demand — the pattern Anthropic measured cutting upfront tool tokens from ~77K to ~8.7K. Three habits:

- **Disconnect servers you are not using this week** rather than denying their tools. Only connected servers cost anything.
- **Scope a noisy server to a subagent** with the `mcpServers` frontmatter field, so its names never reach your main window.
- **Check `/mcp` for per-server cost** before blaming the model.

> **Warning — MCP servers run outside every sandbox tier below the runtime**
>
> Each server is code you run alongside Claude Code. Anthropic does not security-audit third-party servers. Gate them with permission rules, prefer servers you wrote or trust, and remember that `/sandbox` does not cover them — only `@anthropic-ai/sandbox-runtime`, a container, or a VM does. → Ch. 05

## Channels: the outside world knocking

A **channel** is an MCP server that declares `claude/channel` and fires notifications into a _running local session_ — a CI failure, a Telegram message, a webhook. It is the inverse of ordinary MCP, where Claude queries on demand.

| Feature | What it does | Good for |
| --- | --- | --- |
| **Channels** | Push events into your running local session | Chat from your phone; CI into work in progress |
| Claude Code on the web | Fresh cloud sandbox from GitHub | Async tasks you check later |
| Standard MCP | Claude queries on demand during a task | Reading systems when needed |
| Remote Control | Drive a local session from claude.ai or mobile | Steering, not event forwarding |

Two gates must both pass: the server must be in MCP config **and** named on `--channels`, and the sender must be allowlisted. `.mcp.json` alone does not inject.

```
# in Claude Code
/plugin install telegram@claude-plugins-official
/telegram:configure <bot-token>

# restart with the channel active
claude --channels plugin:telegram@claude-plugins-official

# pair, then lock it down
/telegram:access pair <code>
/telegram:access policy allowlist
```

Requires v2.1.80+; permission relay — approving tool use from your phone — requires v2.1.81+. Research preview, Anthropic auth only. `fakechat` runs a localhost UI at `:8787` for testing the pipe without tokens.

> **Warning — Check the sender, not the room**
>
> In group chats `message.from.id` and `message.chat.id` are different fields. Allowlisting the _room_ means anyone in that group can prompt-inject your session. And permission relay means anyone who can reply through the channel can approve tool use — turn it on only when every allowlisted sender is someone you would hand the keyboard.
