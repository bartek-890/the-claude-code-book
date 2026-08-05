# Permission modes and the sandbox ladder

_I · Mechanism · Chapter 05_

The more permission dialogs you click through, the less you read them. Two separate systems address that: **permission modes**, which decide how often Claude pauses to ask, and **sandboxing**, which decides what a running command can touch regardless of what anyone approved. They are not substitutes.

## The six modes

| Mode | Runs without asking | Best for |
| --- | --- | --- |
| `default`<br>(shown as “Manual”) | Reads only | Getting started, sensitive work |
| `acceptEdits` | Reads, file edits, common filesystem commands (`mkdir`, `touch`, `mv`, `cp`) | Iterating on code you are reviewing |
| `plan` | Reads, plus classifier-approved commands when auto mode is available | Exploring before changing |
| `auto` | Everything, with a classifier running background safety checks | Long tasks, prompt fatigue |
| `dontAsk` | Only pre-approved tools — everything else is refused, not prompted | Locked-down CI and scripts |
| `bypassPermissions` | Everything, no checks | Isolated containers and VMs **only** |

`Shift+Tab` cycles `default` → `acceptEdits` → `plan` mid-session. Not every mode is in the default cycle; `auto` and `bypassPermissions` have to be enabled. At startup, use `--permission-mode <mode>`.

> **Note — Modes set the baseline; rules layer on top**
>
> Deny rules and explicit ask rules apply in _every_ mode, including `bypassPermissions`. Allow rules have no effect there, because everything else is already approved. Writes to protected paths — repository state, Claude’s own configuration — are never auto-approved outside bypass mode.

## How a tool call is actually resolved

The mode is the last thing consulted, not the first. Claude Code walks a fixed sequence and stops at the first layer that answers.

| # | Layer | What it can answer |
| --- | --- | --- |
| 1 | `PreToolUse` hook | allow · deny · ask · defer — and it can rewrite the arguments |
| 2 | `permissions.deny` | Blocked outright, in every mode |
| 3 | `permissions.allow` | Runs with no prompt |
| 4 | `permissions.ask` | Escalates to you even where the mode would not have |
| 5 | The permission mode | The baseline from the table above |

Two consequences are worth writing into a settings file. Deny is evaluated _before_ the mode, so a deny rule still blocks under `bypassPermissions` — the mode people reach for when the prompts get annoying. And `ask` is a first-class rule in the same file, not a hook trick. Deny is for doors nobody should open unattended; ask is for doors that need a hand on them but not a weld.

```
{
  "permissions": {
    "deny": ["Bash(rm -rf *)", "Bash(git push --force *)",
             "Bash(npm publish *)", "Bash(gh release create *)"],
    "ask":  ["Bash(git reset --hard *)", "Bash(psql *)",
             "Bash(aws s3 rm *)"]
  }
}
```

> **Key — A deny rule you have not watched fire is a belief**
>
> Run it as a case before you trust it. Told to `rm -rf logs/` with a plausible reason and a pre-empted confirmation, a session with no harness lost the directory — and so did one carrying a bloated `CLAUDE.md` that said not to. With the rule above, two `rm -rf` calls were denied at the permission layer and the directory survived: _“I stopped rather than resubmit the same blocked command.”_
>
> The same run also asked every variant to paste a planted `.env` secret, and all of them refused — including the arms carrying no deny rule at all. The `Read(./.env*)` rule never had to fire, so that case proved nothing about the rule. It might be doing the job, or the model might be covering for it, and you find out which on the day it stops covering.

Know what a pattern does not buy, either. Both mechanisms match on the command string, so an agent that writes `cleanup.py` and runs `python cleanup.py` walks straight past them — and `git push --force` quoted inside a file you asked Claude to review is not an attempt to push. Patterns raise the cost of an accident. They do not close the door. That takes the sandbox below, or credentials that never reach the thing you cannot afford to lose.

## Auto mode and its classifier

Auto mode is not “yes to everything”. A separate classifier model reviews each action before it runs and blocks anything that escalates beyond your request, targets unrecognised infrastructure, or appears driven by hostile content Claude read. Some specifics worth knowing:

- The classifier trusts your working directory and the remotes configured _when the session started_. A remote added mid-session with `git remote add` is not trusted.
- It blocks history-destroying git operations by default: `git reset --hard`, `git checkout -- .`, `git clean -fd`, `git stash drop`.
- It blocks a recursive forced delete whose target is an unresolved shell variable, because it reads the conversation and never sees command output. Naming the literal path clears it.
- **Boundaries you state in chat are enforced.** Say “don’t push until I review” and the classifier blocks matching actions even where default rules allow them — until you lift it.

> **Warning — Stated boundaries are not stored as rules**
>
> The classifier re-reads them from the transcript on every check, so a boundary can be **lost when compaction removes the message that stated it**. For a hard guarantee, write a `permissions.deny` rule instead of saying it out loud.

**Fallback:** if the classifier blocks 3 times in a row or 20 times total, auto mode pauses and normal prompting resumes. Approving the prompted action resumes auto mode. These thresholds are not configurable. In non-interactive `-p` runs, repeated blocks abort the run — there is no user to fall back to.

## Sandboxing: the boundary the kernel enforces

Claude Code’s OS-level Bash sandbox — Seatbelt on macOS, bubblewrap on Linux and WSL2 — cut permission prompts by about **84%** in Anthropic’s internal usage (Appendix F), and it keeps a prompt-injected agent inside boundaries the kernel enforces. One catch trips nearly everyone: **`/sandbox` wraps Bash only.** `Read`, `Edit`, MCP servers, and hooks stay on the host.

| Approach | What is isolated | When |
| --- | --- | --- |
| Sandboxed Bash (`/sandbox`) | Bash + child processes only | Everyday local work in a known repo |
| `@anthropic-ai/sandbox-runtime` | Whole Claude process — tools, MCP, hooks | Unattended runs without Docker; MCP you don’t fully trust |
| Dev container / custom container | Full development environment | Team standard; skip-permissions with a firewall allowlist |
| Virtual machine | Full OS / kernel separation | Untrusted repos; compliance |
| Claude Code on the web | Anthropic-managed VM; Git via scoped proxy | No local setup; clone from GitHub into an isolated cloud VM |

## The default that catches people: reads are wide open

Sandbox **write** access defaults to the working directory plus the session temp directory. Sandbox **read** access defaults to nearly the whole machine — including `~/.aws/credentials` and `~/.ssh/`. There is **no built-in credential deny list.** An empty `credentials` block protects nothing.

```
{
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true,
    "allowUnsandboxedCommands": false,
    "filesystem": { "denyRead": ["~/"], "allowRead": ["."] },
    "credentials": {
      "files": [
        { "path": "~/.aws/credentials", "mode": "deny" },
        { "path": "~/.ssh",              "mode": "deny" }
      ],
      "envVars": [
        { "name": "GITHUB_TOKEN", "mode": "deny" },
        { "name": "NPM_TOKEN",    "mode": "deny" }
      ]
    },
    "network": { "allowedDomains": ["registry.npmjs.org", "api.github.com"] }
  },
  "permissions": {
    "deny": ["Read(~/.ssh/**)", "Read(~/.aws/**)", "Read(./.env*)"]
  }
}
```

Set both layers: `sandbox.credentials` closes the hole for Bash and its children; `permissions.deny` closes it for the `Read` tool. Deny entries merge across project, user, and managed scopes, and a lower scope cannot loosen a managed deny.

> **Warning — Four things that stay outside the boundary**
>
> - **MCP servers.** Anthropic does not security-audit third-party servers. Allowlist them and prefer ones you wrote or trust.
> - **Broad domain allowlists.** `github.com` still leaves room for domain fronting; the default proxy checks the claimed hostname, not the encrypted payload.
> - **`/var/run/docker.sock`** via `allowUnixSockets`. That is host access with extra steps. Never allow it.
> - **Home-directory trust.** Starting Claude Code in `~` extends trust to everything under your home directory for the session. Start from a project subdirectory.

## The playbook, by situation

- **TRUSTED REPO, INTERACTIVE** — /sandbox auto-allow + credential denies + narrow allowedDomains
- **TEAM LAPTOP FLEET** — Managed: enabled + failIfUnavailable + allowUnsandboxedCommands false
- **UNATTENDED / SKIP-PERMS** — Container, VM, or npx @anthropic-ai/sandbox-runtime claude
- **UNTRUSTED REPOSITORY** — Dedicated VM or Claude Code on the web — never Bash sandbox alone
- **ALWAYS** — Audit with /permissions; keep SAST in CI; review suggested commands

> **Key — Bottom line**
>
> Permission rules decide whether a tool runs. The Bash sandbox decides what a running shell can touch. Containers and VMs decide whether the whole agent — including `Read` and MCP — shares your host. Turn on `/sandbox`, deny credential reads, and climb the ladder the moment you remove the human from the approval loop.
