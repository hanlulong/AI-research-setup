# Step 2 — Configure & use effectively (detail)

Get both tools running with the strongest models, the right permission level, and the keyboard shortcuts that matter.

[← Back to README](../README.md)

> [!IMPORTANT]
> **Easy path: ask the AI.** Once Codex or Claude Code is running, paste:
>
> > *"Configure yourself for research work: use the latest model, set reasoning effort to xhigh, keep a sensible permission mode, and add a sensible statusline. Show me the diff before applying."*
>
> Everything below is the reference for what gets changed and where, in case you want to edit by hand.

## Contents

- [Day-1 settings](#day-1-settings)
- [Codex CLI](#codex-cli)
  - [config.toml](#configtoml)
  - [Keyboard shortcuts](#keyboard-shortcuts)
  - [AGENTS.md](#agentsmd)
- [Claude Code](#claude-code)
  - [settings.json](#settingsjson)
  - [Statusline](#statusline)
  - [Permission modes](#permission-modes)
  - [Reasoning, ultracode & fast mode](#reasoning-ultracode--fast-mode)
  - [CLAUDE.md](#claudemd)
- [Skills and MCP servers](#skills-and-mcp-servers)

## Day-1 settings

Out of the box both tools lean cautious — Codex prompts on request, and Claude Code starts in its `default` mode (auto-approves reads, prompts for the rest). For research work, set three things deliberately:

| Setting | Codex CLI | Claude Code |
|---|---|---|
| **File** | `~/.codex/config.toml` | `~/.claude/settings.json` |
| **Latest model** | `model = "gpt-5.5"` | `"model": "claude-opus-4-8"` |
| **Deep reasoning** | `model_reasoning_effort = "xhigh"` | `"effortLevel": "xhigh"` |
| **Permission level** | `approval_policy = "on-request"` (or `"never"` per-project) | `auto` (opt-in — set `"defaultMode": "auto"`) — or `"acceptEdits"` for a tighter leash |

> [!NOTE]
> Model names rotate. Use the `/model` picker inside Codex or Claude Code to see what's available on your plan. As of June 2026, `gpt-5.5` is the Codex default; `claude-opus-4-8` is the strongest Claude.

> [!NOTE]
> **Effort and thinking are separate knobs.** `effortLevel` sets reasoning depth — the ladder runs `low | medium | high | xhigh | max`. Opus 4.8 defaults to `high`; persist `"effortLevel": "xhigh"` for a deeper research default. Separately, `"alwaysThinkingEnabled": true` shows extended thinking by default. For model-specific limits, the deepest settings, and automatic multi-agent workflows, see [Reasoning, ultracode & fast mode](#reasoning-ultracode--fast-mode).

---

## Codex CLI

### config.toml

Two scopes:

- `~/.codex/config.toml` — user-level
- `<repo>/.codex/config.toml` — project-level

Recommended **user-level** `~/.codex/config.toml`:

```toml
model = "gpt-5.5"
model_reasoning_effort = "xhigh"   # minimal | low | medium | high | xhigh (xhigh = deepest)
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

The same depth-vs-speed trade-off applies here: keep `model_reasoning_effort = "xhigh"` for large or correctness-critical work, and switch to the read-only `fast` profile (`codex --profile fast`) for quick, interactive iteration.

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
  "model": "claude-opus-4-8",
  "effortLevel": "xhigh",
  "alwaysThinkingEnabled": true,
  "showThinkingSummaries": true,
  "permissions": {
    "defaultMode": "auto",
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

> [!TIP]
> **Make ultracode your default.** `ultracode` isn't read from your `settings.json` file — make it stick by passing it on every launch via a shell alias instead. It pairs `xhigh` effort with automatic multi-agent workflows on substantive tasks. See [Reasoning, ultracode & fast mode](#reasoning-ultracode--fast-mode).

### Statusline

Claude Code renders a custom status line at the bottom of the TUI — runs an external command, feeds it JSON on stdin (model, cwd, cost, context %, …), prints whatever you want.

Inside Claude Code, run **`/statusline`** to scaffold one interactively. Or ask:

> *"Set up a statusline showing model, current directory, git branch, and session cost."*

<details>
<summary><b>Or write the script yourself</b></summary>

<br>

A starter (Mac/Linux; requires `jq`):

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

`chmod +x ~/.claude/statusline.sh`. On Windows, Claude Code uses Git Bash if installed; for PowerShell, write a `.ps1` and reference it as `powershell -NoProfile -File C:\path\to\statusline.ps1`.

</details>

### Permission modes

`permissions.defaultMode` controls how Claude Code handles tool calls.

| Mode                | What it does                                                                                          |
|---------------------|------------------------------------------------------------------------------------------------------|
| `auto` *(opt-in)*   | Runs tool calls it assesses as lower-risk automatically (after a risk and prompt-injection check) and blocks the rest. |
| `default`           | Prompts the first time each tool is used.                                                             |
| `acceptEdits`       | Auto-accepts file edits and common filesystem commands within the working directory.                 |
| `plan`              | Plan Mode: read-only — Claude inspects and reasons but does not edit.                                 |
| `bypassPermissions` | Skips all prompts. Use only in containers / VMs / scratch dirs.                                       |

**Enable `auto`** — it suits research: it keeps you moving while still pausing on anything risky. It's an opt-in research preview, not the out-of-box default; set `"defaultMode": "auto"` in user settings (see below) to start every session there. Requirements: Claude Code v2.1.83+ and model Opus 4.6 or later (or Sonnet 4.6) on the Anthropic API — on Bedrock / Vertex / Foundry it additionally needs `CLAUDE_CODE_ENABLE_AUTO_MODE=1` and Opus 4.7/4.8. Want a tighter leash on a project? Override to `acceptEdits` or `plan`.

> [!IMPORTANT]
> **Set `auto` at the user level, not in a committed repo file.** `defaultMode: "auto"` is honored only from user (`~/.claude/settings.json`), CLI-flag, or org-policy settings. It is **ignored** in a repo's `<repo>/.claude/settings.json` or `.claude/settings.local.json` — those are repo-controllable, so a cloned repo can't silently grant itself auto. The reverse works fine: a committed `{ "permissions": { "defaultMode": "acceptEdits" } }` (or `"plan"`) *is* respected, so use it to tighten a shared project.

How `auto` decides: it inspects each call for risky or hard-to-reverse actions (root/home removals like `rm -rf ~`, pushes to untrusted destinations, exfiltration-shaped commands) and for prompt injection, runs the lower-risk ones, and blocks the rest. Pre-declare trusted destinations and custom rules under `autoMode.{allow, soft_deny, hard_deny, environment}`.

**Session override:** `claude --permission-mode auto` (or `plan` / `acceptEdits`).

> [!IMPORTANT]
> **Working with sensitive data?** A few habits matter once an AI can read your files and run code:
> - **Prompts and file contents go to the model provider.** Don't paste live credentials or confidential/PII data; load API keys (FRED, Census, …) from environment variables rather than hard-coding them in scripts.
> - **"Install this" and MCP servers run third-party code** with your filesystem access — install only from sources you trust.
> - **For restricted data (IRB / DUA) or embargoed results,** stay in `auto` or `plan` and review actions; save `bypassPermissions` for a throwaway scratch dir.
> - **Leave `auto` on:** its prompt-injection check is what guards against instructions hidden in web pages or PDFs the agent reads.

### Reasoning, ultracode & fast mode

**Effort ladder.** `effortLevel` sets how hard Claude thinks:

| Level    | Use it for                                                     |
|----------|----------------------------------------------------------------|
| `low`    | Quick, mechanical edits.                                       |
| `medium` | Balanced everyday work.                                        |
| `high`   | Comprehensive work with thorough testing — Opus 4.8's default. |
| `xhigh`  | Deeper reasoning, just below maximum. **Opus 4.8/4.7 only.**   |
| `max`    | Maximum capability, deepest reasoning.                         |

Persist `"effortLevel": "xhigh"` in `settings.json` as a durable research default. Bump a single session with `/effort max`, or reset to the model default with `/effort auto`. (`max` is session-only unless you also set `CLAUDE_CODE_EFFORT_LEVEL=max`; on Opus 4.6 and Sonnet 4.6, `xhigh` falls back to `high`.)

**Ultracode — maximum thoroughness.** `/effort ultracode` pairs `xhigh` effort with *standing dynamic-workflow orchestration*: Claude automatically spins up multi-agent workflows for substantive tasks, optimizing for the most exhaustive, correct answer rather than the fastest or cheapest. Ideal for a tricky derivation, a careful refactor, or a thorough audit.

- **Enable for a session:** `/effort ultracode`. Exit with `/effort xhigh` (or `auto`).
- **Make it your default:** launch via a shell alias so every session starts in ultracode:
  ```bash
  alias claude="claude --settings '{\"ultracode\": true}'"
  ```
- **Prerequisites:** an xhigh-capable model (Opus 4.8 or 4.7) and Dynamic Workflows enabled. Workflows default **on** for Max / Team / Enterprise and **off** for Pro — Pro users flip them on in `/config` first.
- **Caveats:** ultracode is session-scoped (not a persistable `settings.json` key) and doesn't apply to remote sessions. It costs more tokens and runs longer — that's the trade for thoroughness.

> [!TIP]
> Typing the literal word **"workflow"** (or "workflows") in a prompt also opts that single prompt into the multi-agent Workflow tool, without changing your effort level.

**Fast mode — the opposite trade-off.** `/fast` (or `"fastMode": true`) runs Claude Opus with ~2.5× faster output — **not** a downgrade to a smaller model — at a higher per-token cost (requires extra usage enabled). Available on Opus 4.8/4.7/4.6; not in the VS Code extension. Use it for quick, interactive iteration; switch back to `xhigh` or ultracode for hard problems.

**Picking a gear.** Reach for **ultracode** on complex or large coding projects — a multi-file refactor, a full replication package or audit, a structural estimation, a subtle derivation — where getting it exhaustively right outweighs speed or token cost. Drop to **`/fast`** or a lower effort level for quick, interactive iteration. Most everyday work sits on the `high`/`xhigh` middle.

> [!NOTE]
> **Codex equivalent.** Codex has no ultracode, but `model_reasoning_effort` (`minimal`…`xhigh`) is its effort dial and `codex --profile fast` its fast lane — see [config.toml](#configtoml).

> [!NOTE]
> **Mind your usage.** Ultracode, fast mode, and several parallel panes all multiply token consumption, and subscription plans enforce rolling usage caps. Watch the statusline's `week %` and reach for the cheaper levers as you near a cap — fewer panes, `/fast`, or a lower effort level — reserving ultracode for the runs that genuinely need it.

**Reviewing code.** Run `/code-review` (effort `low` → `max`) on your changes; `/code-review ultra` launches a deep multi-agent **cloud** review of the current branch or a PR number (user-triggered, billed).

### CLAUDE.md

Project conventions Claude Code reads on every session. Locations:

- `~/.claude/CLAUDE.md` — your global instructions (applies to every project)
- `<repo>/CLAUDE.md` — project instructions, committed to git

Use these for: build/test/lint commands, code style, folder conventions, "always do X / never do Y" rules. Keep them short — every line costs context.

---

## Skills and MCP servers

Both tools support **skills** (reusable instruction bundles invoked with `/<name>` or `$<name>`) and **MCP servers** (external tool capabilities).

**To install either: paste the GitHub URL to your AI and ask "install this."** The AI reads the README and runs the right command for whichever tool you're using.

[docs/software.md](./software.md) lists specific skills and MCP servers worth installing for research work.

<details>
<summary><b>Or run the install commands yourself</b></summary>

<br>

**Claude Code MCP:**

```bash
claude mcp add <name> -- <command> [args...]
# HTTP transport:
claude mcp add --transport http <name> <url> --scope user
```

**Codex CLI MCP:**

```bash
codex mcp add <name> --url <url>
# or add a [mcp_servers.<name>] block in config.toml
```

**Skills:** drop into `~/.claude/skills/<name>/` (Claude Code) or `~/.codex/skills/<name>/` (Codex), or run the skill's own installer.

</details>

→ Next: [Step 3 — Workflow](./workflow.md)
