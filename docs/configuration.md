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

Out of the box, Codex prompts on request, and Claude Code (v2.1.283+) starts interactive sessions in `auto` when the model supports it (otherwise Manual). For research work, set three things deliberately:

| Setting | Codex CLI | Claude Code |
|---|---|---|
| **File** | `~/.codex/config.toml` | `~/.claude/settings.json` |
| **Model** | `model = "gpt-6.1-sol"` | `"model": "claude-opus-5-5"` (alias `opus`) |
| **Deep reasoning** | `model_reasoning_effort = "xhigh"` | `/effort xhigh` (saved per model under `modelSettings`) |
| **Permission level** | `approval_policy = "on-request"` (or `"never"` per-project) | `auto` (built-in start mode since v2.1.283; pin with `"defaultMode": "auto"`), or `"acceptEdits"` for a tighter leash |

> [!NOTE]
> Model names rotate. Use the `/model` picker inside Codex or Claude Code to see what's available on your plan. As of November 2026, `gpt-6.1-sol` is OpenAI's pick for Codex on paid plans and `gpt-6-astra` its most capable (lower usage limits); GPT-5.5 left Codex for ChatGPT sign-in on 2026-10-14. In Claude Code, `opus` = Opus 5.5; Fable 5.1 (`/model fable`) is the most capable for long, hard tasks and may bill to usage credits depending on plan.

> [!NOTE]
> **Effort and thinking are separate knobs.** Effort sets reasoning depth — the ladder runs `low | medium | high | xhigh | max`. Opus 5.5 defaults to `medium`; set `xhigh` with `/effort xhigh`. Thinking is on by default, so no setting is needed. For model-specific limits, the deepest settings, and automatic multi-agent workflows, see [Reasoning, ultracode & fast mode](#reasoning-ultracode--fast-mode).

---

## Codex CLI

### config.toml

Two scopes:

- `~/.codex/config.toml` — user-level
- `<repo>/.codex/config.toml` — project-level

Recommended **user-level** `~/.codex/config.toml`:

```toml
model = "gpt-6.1-sol"   # default pick on Plus/Pro/Business/Enterprise/Edu; hardest work: gpt-6-astra (lower usage limits). Or omit and pick with /model
model_reasoning_effort = "xhigh"   # low | medium | high | xhigh | max | ultra (depends on the model; see /model)
approval_policy        = "on-request"   # on-request | never   ("untrusted" is no longer supported)
sandbox_mode           = "workspace-write"  # read-only | workspace-write | danger-full-access

# Example MCP server: context7 pulls up-to-date library docs on demand
[mcp_servers.context7]
command = "npx"
args    = ["-y", "@upstash/context7-mcp"]
```

For a faster, read-only profile, create `~/.codex/fast.config.toml` with top-level keys only (Codex 0.134+ no longer reads `[profiles.<name>]` tables in config.toml), then run `codex --profile fast`:

```toml
# ~/.codex/fast.config.toml
model_reasoning_effort = "low"
sandbox_mode           = "read-only"
```

The same depth-vs-speed trade-off applies here: keep `model_reasoning_effort = "xhigh"` for large or correctness-critical work. `gpt-6.1-sol`'s own default effort is `low`, so this line is what gives the depth. For quick iteration, use the read-only `fast` profile, or `/fast`, which turns on the model's Fast service tier where it has one and uses 2.5× included usage.

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
  "model": "claude-opus-5-5",
  "effortLevel": "xhigh",
  "modelSettings": { "claude-opus-5-5": { "effortLevel": "xhigh" } },
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

`modelSettings` is what `/effort xhigh` writes for the model you're on (v2.1.251+); typing `/effort xhigh` once per model is the easy way to set it. The top-level `effortLevel` only covers Opus 5, Fable 5.1 and older; Opus 5.5 and newer models ignore it in user settings and start at their own default, so after switching to a new model run `/effort xhigh` again. Anthropic's default for Opus 5.5 is `medium` (which matches or exceeds Opus 5 at `high`); `xhigh` is our research choice and uses noticeably more of your plan.

> [!TIP]
> **Make ultracode your default.** Add `"ultracode": true` to `~/.claude/settings.json` (works on Windows and macOS). It applies to every session and uses noticeably more of your usage. Turn it off for one session with `/effort ultracode off`. Pair it with `xhigh` effort. See [Reasoning, ultracode & fast mode](#reasoning-ultracode--fast-mode).

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

| Mode                                    | What it does                                                                                          |
|-----------------------------------------|------------------------------------------------------------------------------------------------------|
| `auto`                                  | Runs tool calls it assesses as lower-risk automatically (after a risk and prompt-injection check) and blocks the rest. |
| `default` (Manual; `manual` also works) | Runs reads only and asks before edits, shell commands and network access.                             |
| `acceptEdits`                           | Auto-accepts file edits and common filesystem commands within the working directory.                 |
| `plan`                                  | Plan Mode: read-only — Claude inspects and reasons but does not edit.                                 |
| `dontAsk`                               | Runs only pre-approved tools (and reads in your working directories); anything that would prompt is denied. For CI and scripts. |
| `bypassPermissions`                     | Skips all prompts. Use only in containers / VMs / scratch dirs.                                       |

**Keep `auto`.** It suits research: it keeps you moving while still pausing on anything risky. With Claude Code v2.1.283+, interactive terminal and VS Code sessions start there; `claude -p` scripts start in Manual, and a session whose model can't use auto falls back to Manual. Pin it with `"defaultMode": "auto"` in user settings. Model requirements: on the Anthropic API, Opus 4.6+, Sonnet 4.6+, Haiku 5.5 or Fable; on Bedrock, Agent Platform and Foundry, Sonnet 5+, Opus 4.7+, Haiku 5.5 or Fable. Before v2.1.283, auto was the starting mode only on Pro/Max/Ultra/Team, so run `claude update`. Want a tighter leash on a project? Override to `acceptEdits` or `plan`.

> [!IMPORTANT]
> **Set `auto` at the user level, not in a committed repo file.** `defaultMode: "auto"` is honored only from user (`~/.claude/settings.json`), CLI-flag, or org-policy settings. It is **ignored** in a repo's `<repo>/.claude/settings.json` or `.claude/settings.local.json` — those are repo-controllable, so a cloned repo can't silently grant itself auto. `bypassPermissions` is likewise ignored from repo settings. The reverse works fine: a committed `{ "permissions": { "defaultMode": "acceptEdits" } }` (or `"plan"`) *is* respected, so use it to tighten a shared project.

How `auto` decides: it inspects each call for risky or hard-to-reverse actions (root/home removals like `rm -rf ~`, pushes to untrusted destinations, exfiltration-shaped commands) and for prompt injection, runs the lower-risk ones, and blocks the rest. Pre-declare trusted destinations and custom rules under `autoMode.{allow, soft_deny, hard_deny, environment}`.

**Session override:** `claude --permission-mode auto` (or `plan` / `acceptEdits`).

> [!IMPORTANT]
> **Working with sensitive data?** A few habits matter once an AI can read your files and run code:
> - **Prompts and file contents go to the model provider.** Don't paste live credentials or confidential/PII data; load API keys (FRED, Census, …) from environment variables rather than hard-coding them in scripts.
> - **"Install this" and MCP servers run third-party code** with your filesystem access — install only from sources you trust.
> - **For restricted data (IRB / DUA) or embargoed results,** stay in `auto`, `plan`, or Manual (`"defaultMode": "default"`, which asks before edits and commands) and review actions; save `bypassPermissions` for a throwaway scratch dir.
> - **Leave `auto` on:** its prompt-injection check is what guards against instructions hidden in web pages or PDFs the agent reads.

### Reasoning, ultracode & fast mode

**Effort ladder.** The effort level (`/effort`) sets how hard Claude thinks:

| Level    | Use it for                                                     |
|----------|----------------------------------------------------------------|
| `low`    | Quick, mechanical edits.                                       |
| `medium` | Balanced everyday work.                                        |
| `high`   | Comprehensive work with thorough testing; the default on most models (Opus/Sonnet/Haiku 5.5 default to `medium`, Opus 4.7 to `xhigh`). |
| `xhigh`  | Deeper reasoning, just below maximum. Opus 5.5/5, Sonnet 5.5/5, Haiku 5.5, Fable, Opus 4.8/4.7. |
| `max`    | Maximum capability, deepest reasoning.                         |

Run `/effort xhigh` to save `xhigh` for the model you're on (saved per model; a new model starts at its own default). Bump a single session with `/effort max`, or clear your saved level with `/effort auto`. (`max` is session-only unless you set `CLAUDE_CODE_EFFORT_LEVEL=max`; on a model without `xhigh`, such as Opus 4.6, `xhigh` falls back to `high`.)

**Ultracode — maximum thoroughness.** Ultracode is a toggle, not an effort level. With it on, Claude automatically orchestrates multi-agent dynamic workflows for substantive tasks, at whatever effort the session runs at (v2.1.284+), so pair it with `xhigh`. Ideal for a tricky derivation, a careful refactor, or a thorough audit.

- **Session:** `/effort ultracode`, off with `/effort ultracode off`; picking another effort level leaves it on. `claude --effort ultracode` launches with it on at `xhigh`.
- **Default:** the `"ultracode": true` setting (see the tip under [settings.json](#settingsjson)).
- **Prerequisites:** an xhigh-capable model, and Dynamic workflows (all paid plans; on Pro, turn them on in `/config`).
- **Caveat:** every request uses more tokens and takes longer, so you reach session or weekly limits sooner.

> [!TIP]
> Include the keyword `ultracode` in a prompt (it's highlighted as you type), or ask for a workflow in plain words, to run that one prompt as a multi-agent workflow without changing your effort level. Turn the keyword off with "Ultracode keyword trigger" in `/config`.

**Fast mode — the opposite trade-off.** `/fast` (or `"fastMode": true`) runs Claude Opus up to 2.5× faster — **not** a smaller model — at a higher per-token cost. Supported on Opus 5.5 and Opus 5; the VS Code extension has a "Toggle fast mode" command. On subscription plans it bills to usage credits only, even when you have plan usage left. Use it when speed is worth paying for; switch back for hard problems.

**Picking a gear.** Reach for **ultracode** on complex or large coding projects — a multi-file refactor, a full replication package or audit, a structural estimation, a subtle derivation — where getting it exhaustively right outweighs speed or token cost. Drop to a lower effort level (or `/fast` when speed is worth paying usage credits for) for quick, interactive iteration. Most everyday work sits on the `high`/`xhigh` middle.

> [!NOTE]
> **Codex equivalent.** Codex has no ultracode setting. `model_reasoning_effort` is its effort dial (`low` … `max`). `ultra`, on models that offer it, splits a task across subagents, the closest analogue to ultracode. For speed: `/fast` (Fast tier, 2.5× included usage) or the read-only `fast` profile — see [config.toml](#configtoml).

> [!NOTE]
> **Mind your usage.** Ultracode, fast mode, and several parallel panes all multiply token consumption, and subscription plans enforce rolling usage caps. Watch the statusline's `week %` and reach for the cheaper levers as you near a cap — fewer panes or a lower effort level — reserving ultracode for the runs that genuinely need it. Fast mode isn't a cheaper lever: on subscriptions it draws on usage credits.

**Reviewing code.** Run `/code-review` (effort `low` → `max`) on your changes; `/code-review ultra` launches a deep multi-agent **cloud** review of the current branch or a PR number (user-triggered, billed). Codex's equivalent is `/review`.

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
