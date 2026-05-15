# Step 2 — Configure & use effectively (detail)

Get both tools running with the strongest models, the right permission level, and the keyboard shortcuts that matter.

[← Back to README](../README.md)

> [!IMPORTANT]
> **Easy path: ask the AI.** Once Codex or Claude Code is running, paste:
>
> > *"Configure yourself for research work: use the latest model, max thinking effort, acceptEdits permissions, and a sensible statusline. Show me the diff before applying."*
>
> Everything below is the reference for what gets changed and where, in case you want to edit by hand.

## Contents

- [Day-1 settings](#day-1-settings)
- [Claude Code](#claude-code)
  - [settings.json](#settingsjson)
  - [Statusline](#statusline)
  - [Permission modes](#permission-modes)
  - [CLAUDE.md](#claudemd)
- [Codex CLI](#codex-cli)
  - [config.toml](#configtoml)
  - [Keyboard shortcuts](#keyboard-shortcuts)
  - [AGENTS.md](#agentsmd)
- [Skills and MCP servers](#skills-and-mcp-servers)

## Day-1 settings

Both tools default to caution. For research work, opt into three things:

| | Codex CLI | Claude Code |
|---|---|---|
| **File** | `~/.codex/config.toml` | `~/.claude/settings.json` |
| **Latest model** | `model = "gpt-5.5"` | `"model": "claude-opus-4-7"` |
| **Max thinking** | `model_reasoning_effort = "xhigh"` | `"effortLevel": "xhigh"`, `"alwaysThinkingEnabled": true` |
| **Skip routine prompts** | `approval_policy = "on-request"` (or `"never"` per-project) | `"permissions": { "defaultMode": "acceptEdits" }` |

> [!NOTE]
> Model names rotate. Run `codex --list-models` and check `/model` inside Claude Code for what's currently available on your plan. As of May 2026, `gpt-5.5` is the Codex default; `claude-opus-4-7` is the strongest Claude.

---

## Claude Code

### settings.json

Three scopes, in precedence order (later overrides earlier):

- `~/.claude/settings.json` — user-level, all projects
- `<repo>/.claude/settings.json` — project-level, committed to git
- `<repo>/.claude/settings.local.json` — project-local, gitignored

Recommended **user-level** `~/.claude/settings.json`:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "claude-opus-4-7",
  "alwaysThinkingEnabled": true,
  "showThinkingSummaries": true,
  "effortLevel": "xhigh",
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": [
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(ls:*)"
    ]
  },
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh"
  }
}
```

### Statusline

Claude Code can render a custom status line at the bottom of the TUI. It runs an external command, feeds it JSON on stdin (model, cwd, cost, context-window usage, …), and prints whatever you write to stdout.

Configure via the `statusLine` key (already in the snippet above). A starter script (Mac/Linux; requires `jq`):

```bash
#!/usr/bin/env bash
# ~/.claude/statusline.sh
input=$(cat)
MODEL=$(echo "$input" | jq -r '.model.display_name // "claude"')
DIR=$(echo "$input"   | jq -r '.workspace.current_dir // .cwd // "."')
COST=$(echo "$input"  | jq -r '.cost.total_cost_usd // 0')
BRANCH=$(cd "$DIR" 2>/dev/null && git rev-parse --abbrev-ref HEAD 2>/dev/null)
WEEK=$(echo "$input"  | jq -r '.rate_limits.seven_day.used_percentage // empty')

printf "[%s] %s%s  \$%.2f" \
  "$MODEL" \
  "$(basename "$DIR")" \
  "${BRANCH:+ ($BRANCH)}" \
  "$COST"

[ -n "$WEEK" ] && printf "  | week %.0f%%" "$WEEK"
```

Don't forget `chmod +x ~/.claude/statusline.sh`. Inside Claude Code, the `/statusline` command can scaffold one interactively too.

**Windows:** Claude Code uses Git Bash if installed, else PowerShell. The bash script above works under Git Bash; for PowerShell, write a `.ps1` and reference it as `powershell -NoProfile -File C:\path\to\statusline.ps1`.

### Permission modes

`permissions.defaultMode` controls how Claude Code asks before running tools.

| Mode                | What it does                                                                                          |
|---------------------|-------------------------------------------------------------------------------------------------------|
| `default`           | Prompts the first time each tool is used.                                                             |
| `acceptEdits`       | Auto-accepts file edits and common filesystem commands within the working directory.                  |
| `plan`              | Plan Mode: read-only — Claude inspects and reasons but does not edit.                                 |
| `auto`              | Auto-approves tool calls with a background safety classifier blocking destructive / exfiltration ops. |
| `bypassPermissions` | Skips all prompts. Use only in containers / VMs / scratch dirs.                                       |

For research work, start at `acceptEdits` user-wide. Opt into `auto` **per-project** in `<repo>/.claude/settings.json` once you trust the repo's blast radius:

```json
{ "permissions": { "defaultMode": "auto" } }
```

What `auto` still blocks: root/home removals (`rm -rf /`, `rm -rf ~`), pushes to untrusted external domains, obvious data-exfiltration patterns. Trusted destinations go under `autoMode.environment`.

**Session override:** `claude --permission-mode auto` or `claude --permission-mode plan`.

### CLAUDE.md

Project conventions Claude Code reads on every session. Locations:

- `~/.claude/CLAUDE.md` — your global instructions (applies to every project)
- `<repo>/CLAUDE.md` — project instructions, committed to git

Use these for: build/test/lint commands, code style, folder conventions, "always do X / never do Y" rules. Keep them short — every line costs context.

---

## Codex CLI

### config.toml

Two scopes:

- `~/.codex/config.toml` — user-level
- `<repo>/.codex/config.toml` — project-level

Recommended **user-level** `~/.codex/config.toml`:

```toml
model = "gpt-5.5"
model_reasoning_effort = "xhigh"   # none | minimal | low | medium | high | xhigh
approval_policy        = "on-request"   # untrusted | on-request | never
sandbox_mode           = "workspace-write"  # read-only | workspace-write | danger-full-access

# Example MCP server: context7 pulls up-to-date library docs on demand
[mcp_servers.context7]
command = "npx"
args    = ["-y", "@upstash/context7-mcp"]

# A faster, read-only profile — switch with: codex --profile fast
[profiles.fast]
model_reasoning_effort = "low"
sandbox_mode           = "read-only"
```

### Keyboard shortcuts

The non-obvious one — **Tab vs Enter** in the TUI composer:

- **Enter** sends your message immediately. If Codex is mid-turn, Enter *injects* your message into the running turn (interrupts/redirects).
- **Tab** *queues* your typed text for the next turn. Codex picks it up after the current turn finishes, without interruption.

Use Tab to add context while Codex works. Use Enter to course-correct mid-flight.

Other essential keys:

| Key       | Action                                          |
|-----------|-------------------------------------------------|
| `Ctrl+C`  | Close session                                   |
| `Ctrl+L`  | Clear screen (keep history)                     |
| `Ctrl+O`  | Copy latest completed output                    |
| `Ctrl+R`  | Search prompt history                           |
| `Ctrl+G`  | Open `$EDITOR` to draft a longer prompt         |
| `Esc Esc` | Edit your previous message (tap again for older)|
| `↑` / `↓` | Navigate composer draft history                 |

### AGENTS.md

Codex's equivalent of CLAUDE.md. Project conventions, read on every session. Scaffold one with `/init` inside Codex.

---

## Skills and MCP servers

Both tools support **skills** (reusable instruction bundles invoked with `/<name>`) and **MCP servers** (external tool capabilities).

**Install a skill:** drop into `~/.claude/skills/<name>/` or run the skill's installer (e.g. the [econ-writing-skill](https://github.com/hanlulong/econ-writing-skill) curl one-liner — see [docs/software.md](./software.md)).

**Install an MCP server:**

- Claude Code:
  ```bash
  claude mcp add <name> -- <command> [args...]
  # or for HTTP-transport servers:
  claude mcp add --transport http <name> <url> --scope user
  ```
- Codex CLI: add a `[mcp_servers.<name>]` block to `config.toml`, or:
  ```bash
  codex mcp add <name> --url <url>
  ```

[docs/software.md](./software.md) covers specific MCP servers worth installing.

→ Next: [Step 3 — Workflow](./workflow.md)
