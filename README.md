# AI Research Setup

Tips and notes for setting up AI tools and workflows for research work, on **Windows** and **macOS**.

This is a living collection. Each section captures what I've found useful — installation steps, configuration, and gotchas — for the tools I actually use day-to-day.

> These tools update fast. When in doubt, the linked official docs win.

## Contents

- [Windows](#windows)
  - [Codex CLI](#codex-cli-windows)
  - [Claude Code](#claude-code-windows)
- [Mac](#mac)
  - [Codex CLI](#codex-cli-mac)
  - [Claude Code](#claude-code-mac)
- [Shared](#shared)
- [License](#license)

---

## Windows

Tested on Windows 11. Most steps assume **Windows Terminal** with either PowerShell 7 or CMD. WSL2 is fine too and is called out where it differs.

### <a id="codex-cli-windows"></a>Codex CLI

[Codex CLI](https://github.com/openai/codex) is OpenAI's terminal coding agent.

#### Install

Native Windows is supported — WSL is no longer required. Pick one:

```powershell
# Recommended: winget (native)
winget install -e --id OpenAI.Codex

# Or: npm (cross-platform, needs Node.js 22+)
npm install -g @openai/codex

# Or: inside WSL2 Ubuntu
npm install -g @openai/codex
```

Then verify:

```powershell
codex --version
```

**Windows gotchas:**

- **VC++ runtime** is required by the npm build. If `codex` exits silently, install it: `winget install -e --id Microsoft.VCRedist.2015+.x64`.
- **Binary name** from winget can land as `codex-x86_64-pc-windows-msvc.exe`. If `codex` isn't found on PATH, alias it or rename.
- **`rg` on PATH:** Codex shells out to ripgrep. winget installs it under `%LOCALAPPDATA%\Microsoft\WinGet\Links` — make sure that's on your `PATH`.

#### Configure

First-run authentication:

```powershell
codex
```

The TUI launches and prompts you to **Sign in with ChatGPT** (browser OAuth). This is the default and uses your ChatGPT Plus/Pro/Business plan — no separate API billing. To use an API key instead, set `OPENAI_API_KEY` in your environment before running `codex`.

Config lives at `%USERPROFILE%\.codex\config.toml`. A minimal researcher-friendly starter:

```toml
# %USERPROFILE%\.codex\config.toml

approval_policy = "on-request"      # untrusted | on-request | never
sandbox_mode    = "workspace-write" # read-only | workspace-write | danger-full-access

# Pick a model — list available with: codex --list-models
# model = "gpt-5.5"
# model_reasoning_effort = "high"

# Example MCP server: context7 pulls up-to-date library docs on demand
[mcp_servers.context7]
command = "npx"
args    = ["-y", "@upstash/context7-mcp"]

# A faster, read-only profile — switch with: codex --profile fast
[profiles.fast]
sandbox_mode = "read-only"
model_reasoning_effort = "low"
```

Drop a project-level `AGENTS.md` at the repo root (`/init` inside the TUI scaffolds one) so Codex picks up project conventions automatically.

Useful slash commands inside Codex: `/init`, `/resume`, `/model`, `/permissions`, `/plan`, `/diff`, `/mcp`, `/status`.

### <a id="claude-code-windows"></a>Claude Code

[Claude Code](https://docs.claude.com/en/docs/claude-code) is Anthropic's terminal coding agent.

#### Install

Native Windows is the recommended path (no WSL needed).

```powershell
# PowerShell
irm https://claude.ai/install.ps1 | iex
```

```bat
:: CMD
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

Recommended companion: [**Git for Windows**](https://git-scm.com/downloads/win) — Claude Code uses Git Bash as its shell when present, which gives you a Unix-like environment for shell commands. Without it, it falls back to PowerShell.

Verify:

```powershell
claude --version
```

**Windows gotchas:**

- **Pick the right install command for your shell:** PowerShell prompts look like `PS C:\...>`; CMD looks like `C:\...>` (no `PS`).
- **Git Bash not auto-detected?** Point Claude Code at it explicitly in `~/.claude/settings.json`:
  ```json
  {
    "env": {
      "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe"
    }
  }
  ```
- The native installer **auto-updates** in the background. No `npm update -g` needed.

#### Configure

First run:

```powershell
claude
```

It opens a browser window for authentication against your Anthropic (Claude.ai) account — Pro, Max, Team, or Enterprise. No API key required for that path. If you'd rather use API billing, set `ANTHROPIC_API_KEY`.

Config files (precedence: project-local > project > user):

- `%USERPROFILE%\.claude\settings.json` — user-level, all projects
- `<repo>\.claude\settings.json` — project-level, committed to git
- `<repo>\.claude\settings.local.json` — project-local, gitignored
- `%USERPROFILE%\.claude\CLAUDE.md` — your global instructions
- `<repo>\CLAUDE.md` — project instructions, committed

Minimal user `settings.json`:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "claude-sonnet-4-6",
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(ls:*)"
    ]
  }
}
```

What's worth setting on day one:

- **`model`** — `claude-sonnet-4-6` is a balanced default; switch to `claude-opus-4-7` for hardest work or `claude-haiku-4-5` for fast/cheap.
- **`permissions.allow`** — pre-allowlist read-only Bash commands you run constantly so you stop getting prompted. The `/fewer-permission-prompts` skill can scan your transcripts and propose a list.
- **`CLAUDE.md`** — keep one at each repo root with build/test/lint commands and any project-specific conventions.

Add an MCP server:

```powershell
claude mcp add context7 -- npx -y @upstash/context7-mcp
```

A few first-10-minutes commands: `/help`, `/config`, `/resume`, `/compact`, `/memory`, `/skills`.

---

## Mac

Tested on Apple Silicon (macOS 14+). Intel Macs work too unless noted.

### <a id="codex-cli-mac"></a>Codex CLI

#### Install

```bash
# Recommended
brew install --cask codex

# Or via npm (needs Node.js 22+)
npm install -g @openai/codex
```

Requires macOS 12+. The Homebrew cask auto-picks the right architecture (Apple Silicon or Intel).

Verify:

```bash
codex --version
```

#### Configure

First run:

```bash
codex
```

Authentication and config conventions match the [Windows section above](#codex-cli-windows) — same OAuth flow, same `config.toml` schema. The only difference is the path:

- Config lives at `~/.codex/config.toml`.

Minimal starter (same as Windows, with the unix path):

```toml
# ~/.codex/config.toml

approval_policy = "on-request"
sandbox_mode    = "workspace-write"

[mcp_servers.context7]
command = "npx"
args    = ["-y", "@upstash/context7-mcp"]

[profiles.fast]
sandbox_mode = "read-only"
model_reasoning_effort = "low"
```

### <a id="claude-code-mac"></a>Claude Code

#### Install

```bash
# Recommended: native installer
curl -fsSL https://claude.ai/install.sh | bash
```

Auto-updates in background, no Node.js required, universal binary (Apple Silicon + Intel).

Alternatives:

```bash
# Homebrew, stable channel (lags by about a week)
brew install --cask claude-code

# Homebrew, rolling latest
brew install --cask claude-code@latest
```

If you use Homebrew, you update manually with `brew upgrade claude-code`.

Verify:

```bash
claude --version
```

#### Configure

First run:

```bash
claude
```

Same browser-auth flow as Windows. Config file conventions are identical — only the paths change:

- `~/.claude/settings.json` — user
- `<repo>/.claude/settings.json` — project (committed)
- `<repo>/.claude/settings.local.json` — project-local (gitignored)
- `~/.claude/CLAUDE.md` — your global instructions
- `<repo>/CLAUDE.md` — project instructions

Minimal `~/.claude/settings.json`:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "claude-sonnet-4-6",
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(ls:*)"
    ]
  }
}
```

Add an MCP server:

```bash
claude mcp add context7 -- npx -y @upstash/context7-mcp
```

See the [Windows configure section](#claude-code-windows) for the rest — model choice notes, `CLAUDE.md` conventions, first-10-minutes slash commands.

---

## Shared

*Cross-platform tips and workflows — to be filled in as the repo grows.*

- Editors with AI integration (Cursor, VS Code, JetBrains)
- MCP servers worth running
- Prompts and workflow patterns
- Git, GitHub, and review workflows

---

## License

[MIT](LICENSE)
