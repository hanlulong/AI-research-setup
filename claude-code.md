# Claude Code configuration

Common settings worth turning on once you're past the first day with Claude Code. Everything here goes in `~/.claude/settings.json` (user-level) unless noted. Project-level overrides live at `<repo>/.claude/settings.json` (committed) or `<repo>/.claude/settings.local.json` (gitignored).

[← Back to README](./README.md)

## Contents

- [Statusline](#statusline)
- [Permission modes (including `auto`)](#permission-modes)
- [Extended thinking & effort](#extended-thinking)
- [Putting it together](#full-example)

---

## <a id="statusline"></a>Statusline

Claude Code can render a custom status line at the bottom of the TUI. It runs an external command, feeds it JSON on stdin (model, cwd, cost, context-window usage, …), and prints whatever you write to stdout.

Configure it via the `statusLine` key:

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh"
  }
}
```

A starter script (Mac/Linux; requires `jq`):

```bash
#!/usr/bin/env bash
# ~/.claude/statusline.sh
input=$(cat)
MODEL=$(echo "$input" | jq -r '.model.display_name // "claude"')
DIR=$(echo "$input"   | jq -r '.workspace.current_dir // .cwd // "."')
COST=$(echo "$input"  | jq -r '.cost.total_cost_usd // 0')
BRANCH=$(cd "$DIR" 2>/dev/null && git rev-parse --abbrev-ref HEAD 2>/dev/null)

printf "[%s] %s%s  \$%.2f" \
  "$MODEL" \
  "$(basename "$DIR")" \
  "${BRANCH:+ ($BRANCH)}" \
  "$COST"
```

Don't forget `chmod +x ~/.claude/statusline.sh`.

**Windows note:** Claude Code's shell is Git Bash if installed, else PowerShell. The bash script above works under Git Bash. For PowerShell, write a `.ps1` and reference it as `powershell -NoProfile -File C:\path\to\statusline.ps1`.

Inside Claude Code, the `/statusline` command can also scaffold one interactively if you'd rather not write the script by hand.

---

## <a id="permission-modes"></a>Permission modes

`permissions.defaultMode` controls how Claude Code asks before running tools.

| Mode                 | What it does                                                                                          |
|----------------------|-------------------------------------------------------------------------------------------------------|
| `default`            | Prompts the first time each tool is used.                                                             |
| `acceptEdits`        | Auto-accepts file edits and common filesystem commands within the working directory.                  |
| `plan`               | Plan Mode: read-only — Claude can inspect files and run read-only shell commands but cannot edit.     |
| `auto`               | Auto-approves tool calls with a background safety classifier blocking destructive / exfiltration ops. |
| `bypassPermissions`  | Skips all prompts. Only safe in containers / VMs / scratch dirs.                                      |

To opt into the **auto** mode globally:

```json
{
  "permissions": {
    "defaultMode": "auto"
  }
}
```

What `auto` mode still blocks: removals of root/home (`rm -rf /`, `rm -rf ~`), pushes to untrusted external domains, obvious data-exfiltration patterns. It works best when you also configure trusted destinations under `autoMode.environment` (e.g., your GitHub orgs and your cloud buckets) — see the [official permissions docs](https://docs.claude.com/en/docs/claude-code/iam#auto-mode).

**One-off override:** instead of editing settings, pass `--permission-mode auto` when launching `claude`. There's also `--dangerously-skip-permissions` which maps to `bypassPermissions` mode — use it only on disposable environments.

**Recommendation:** keep your user-level default at `default` or `acceptEdits`, and turn on `auto` per-project (in `<repo>/.claude/settings.json`) once you trust the repo's blast radius. Public repos with no remote access? Safe. Anything that can push to prod or send email? Stay on `acceptEdits`.

---

## <a id="extended-thinking"></a>Extended thinking & effort

For research-grade work — debugging tricky bugs, designing experiments, proof writing — turning on extended thinking and raising the effort level meaningfully improves output. These keys live at the top of `settings.json`:

```json
{
  "alwaysThinkingEnabled": true,
  "showThinkingSummaries": true,
  "effortLevel": "high"
}
```

- **`alwaysThinkingEnabled`** — Claude reasons through harder problems before answering.
- **`showThinkingSummaries`** — surfaces a short summary of the reasoning in the TUI (helpful for spotting wrong directions early).
- **`effortLevel`** — `"low" | "medium" | "high" | "xhigh"`. Higher = better answers, slower, more tokens. Pick based on the task; you can also override per-session.

These aren't "experimental" anymore but they're off by default. For most researcher workflows, `high` is the right place to live.

A handful of features genuinely sit behind feature flags (env vars like `CLAUDE_CODE_EXPERIMENTAL_*` and an `experimental` settings block). Names rotate often — check `claude --help` and the [settings reference](https://docs.claude.com/en/docs/claude-code/settings) for the current set rather than hardcoding flags from a snapshot.

---

## <a id="full-example"></a>Putting it together

A reasonable user-level `~/.claude/settings.json` for daily research work:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "claude-sonnet-4-6",
  "alwaysThinkingEnabled": true,
  "showThinkingSummaries": true,
  "effortLevel": "high",
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

Then, per-project where you're comfortable letting Claude run without prompts:

```json
{
  "permissions": {
    "defaultMode": "auto"
  }
}
```

That gives you: extended thinking everywhere, no prompts on routine edits, a sane status line, and full auto only on repos you've explicitly opted into.
