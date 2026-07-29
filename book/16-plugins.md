# Plugins

_III · Harness · Chapter 16_

A plugin is the distribution format for everything else in this part. Skills, subagents, hooks, MCP servers, LSP servers, and background monitors, bundled into one installable, versioned directory. Nothing in a plugin is a new capability — the value is that a teammate installs it with one command instead of copying six files into the right places.

## Standalone or plugin?

| Approach | Skill names | Best for |
| --- | --- | --- |
| **Standalone** — a `.claude/` directory | `/hello` | Personal workflows, project-specific customisation, experiments |
| **Plugin** — a directory with a manifest | `/plugin-name:hello` | Sharing with a team, distributing, versioned releases, reuse across projects |

The sensible order is **standalone first, plugin when you are ready to share.** Start in `.claude/` for the short names and fast iteration; convert once a second person wants it. Namespacing is the trade: plugin skills are always prefixed, which prevents collisions between plugins but costs you the bare `/deploy`.

## Structure

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json   # manifest — ONLY this file goes in here
├── skills/           # <name>/SKILL.md directories
├── agents/           # subagent definitions
├── hooks/hooks.json  # the same `hooks` object as settings.json
├── .mcp.json         # MCP servers
├── .lsp.json         # language servers for code intelligence
├── monitors/         # background watchers (monitors.json)
├── bin/              # executables added to Bash's PATH while enabled
└── settings.json     # default settings when enabled
```

> **Warning — The mistake everyone makes once**
>
> **Do not put `skills/`, `agents/`, `commands/`, or `hooks/` inside `.claude-plugin/`.** Only `plugin.json` goes there; everything else sits at the plugin root. And the plugin root is the plugin’s own directory — never `~/.claude/`, so a `.mcp.json` placed at `~/.claude/.mcp.json` is simply not read.

The manifest is four fields, of which one matters most:

```
{
  "name": "my-first-plugin",        # also the skill namespace
  "description": "…",               # shown in the plugin manager
  "version": "1.0.0",               # omit and every commit is a new version
  "author": { "name": "You" }
}
```

**Set `version`.** Without it, a git-distributed plugin uses the commit SHA, so every commit you push counts as a release for everyone who installed it.

## Develop and test

```
# load a local plugin for one session — also accepts a .zip (v2.1.128+)
claude --plugin-dir ./my-plugin

# several at once
claude --plugin-dir ./one --plugin-dir ./two

# fetch a packaged archive, e.g. a CI build artifact
claude --plugin-url https://example.com/my-plugin.zip

# scaffold one that auto-loads from your skills directory
claude plugin init my-tool

# validate before you distribute — the review pipeline runs the same check
claude plugin validate
```

`/reload-plugins` picks up changes without restarting — skills, agents, hooks, plugin MCP servers, and plugin LSP servers all reload. A `--plugin-dir` plugin overrides an installed one of the same name for that session, which is how you test a change to something you already have installed.

> **Note — The lightest path of all**
>
> `claude plugin init my-tool` creates `~/.claude/skills/my-tool/` with a manifest and a starter `SKILL.md`. It loads on the next session as `my-tool@skills-dir` — no marketplace, no install step, no flag. Adding a `.claude-plugin/plugin.json` to any existing skill folder does the same thing, and lets that folder bundle agents, hooks, and MCP servers.

## Installing and marketplaces

`/plugin` opens the manager. Two public marketplaces exist: **`claude-plugins-official`**, curated by Anthropic and registered automatically on first interactive launch, and **`claude-community`**, where third-party submissions land after review — added with `/plugin marketplace add anthropics/claude-plugins-community`. Teams host their own marketplace in a private repository.

The highest-value install for most people is a **code-intelligence (LSP) plugin** for their language. It gives Claude real symbol navigation and automatic error detection after edits, which is categorically better than grep for “who calls this?” and removes a whole class of hallucinated API usage.

## Governance

| Setting | Effect |
| --- | --- |
| `strictKnownMarketplaces` (managed) | Restrict which marketplace sources are allowed |
| `disableSideloadFlags` (managed) | Reject `--plugin-dir` / `--plugin-url` / MCP override flags |
| Force-enable / force-disable | Managed settings win — `--plugin-dir` cannot override them |

> **Warning — A plugin is code you are running**
>
> It can ship hooks that execute on every tool call, MCP servers that talk to the network, and executables added to Bash’s `PATH`. The same trust question applies as to any dependency: who wrote it, can you read it, and would you run it if it were a `postinstall` script? Community-marketplace plugins pass automated safety screening and are pinned to a reviewed commit; a random `--plugin-url` is not.

## Migrating from `.claude/`

1. **Create the manifest**

   `mkdir -p my-plugin/.claude-plugin`, then write `plugin.json`.

2. **Copy skills, agents, and commands to the plugin root**

   `cp -r .claude/skills my-plugin/` and so on. Directory names stay the same.

3. **Move hooks into `hooks/hooks.json`**

   The format is identical to the `hooks` object in `settings.json`, so it is a copy-paste. Hook input arrives as JSON on stdin — use `jq` to pull the file path out.

   ```
   { "type": "command",
     "command": "jq -r '.tool_input.file_path' | xargs npm run lint:fix" }
   ```

4. **Test, then delete the originals**

   Project and user `.claude/agents/` definitions **override same-named plugin agents**, so the plugin version only takes effect once the originals are gone. Skills are different: both the bare `/name` and the namespaced `/plugin-name:name` stay available side by side.
