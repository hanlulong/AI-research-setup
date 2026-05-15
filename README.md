# AI Research Setup

Tips and notes for setting up AI tools and workflows for research work, on **Windows** and **macOS**.

This is a living collection. Each section captures what I've found useful — installation steps, configuration, and gotchas — for the tools I actually use day-to-day.

> These tools update fast. When in doubt, the linked official docs win.

## Contents

- [Windows](#windows)
  - [Prerequisites](#windows-prereqs)
  - [Codex CLI](#codex-cli-windows)
  - [Claude Code](#claude-code-windows)
- [Mac](#mac)
  - [Prerequisites](#mac-prereqs)
  - [Codex CLI](#codex-cli-mac)
  - [Claude Code](#claude-code-mac)
- [Shared](#shared)
- [License](#license)

---

## Windows

Tested on Windows 11. Most steps assume **Windows Terminal** with either PowerShell 7 or CMD. WSL2 is fine too.

### <a id="windows-prereqs"></a>Prerequisites

Both tools are installed via npm, so you need **Node.js (LTS, v22+)**. The easiest path is winget:

```powershell
winget install -e --id OpenJS.NodeJS.LTS
```

Alternatives: the official installer from [nodejs.org](https://nodejs.org/), or a version manager like [nvm-windows](https://github.com/coreybutler/nvm-windows) / [fnm](https://github.com/Schniz/fnm) if you want multiple Node versions side-by-side.

Verify:

```powershell
node --version   # should print v22.x or higher
npm --version
```

One more component, recommended for Codex CLI on Windows:

```powershell
winget install -e --id Microsoft.VCRedist.2015+.x64
```

Without the VC++ runtime, `codex` can exit silently after install ([issue #20827](https://github.com/openai/codex/issues/20827)).

<a id="codex-cli-windows"></a>
<details>
<summary><b>Codex CLI</b> — OpenAI's terminal coding agent (<a href="https://github.com/openai/codex">repo</a>)</summary>

<br>

#### Install

```powershell
npm install -g @openai/codex
```

Verify:

```powershell
codex --version
```

To update later:

```powershell
npm update -g @openai/codex
```

#### Configure

First run:

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

</details>

<a id="claude-code-windows"></a>
<details>
<summary><b>Claude Code</b> — Anthropic's terminal coding agent (<a href="https://docs.claude.com/en/docs/claude-code">docs</a>)</summary>

<br>

#### Install

```powershell
npm install -g @anthropic-ai/claude-code
```

Recommended companion: [**Git for Windows**](https://git-scm.com/downloads/win) — Claude Code uses Git Bash as its shell when present, which gives you a Unix-like environment for shell commands. Without it, it falls back to PowerShell.

Verify:

```powershell
claude --version
```

To update later:

```powershell
npm update -g @anthropic-ai/claude-code
```

**Windows gotchas:**

- **Git Bash not auto-detected?** Point Claude Code at it explicitly in `~/.claude/settings.json`:
  ```json
  {
    "env": {
      "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe"
    }
  }
  ```

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

</details>

---

## Mac

Tested on Apple Silicon (macOS 14+). Intel Macs work too unless noted.

### <a id="mac-prereqs"></a>Prerequisites

Both tools are installed via npm, so you need **Node.js (LTS, v22+)**. The easiest path is [Homebrew](https://brew.sh/):

```bash
brew install node
```

Alternatives: the official installer from [nodejs.org](https://nodejs.org/), or a version manager like [nvm](https://github.com/nvm-sh/nvm) / [fnm](https://github.com/Schniz/fnm) if you want multiple Node versions side-by-side.

Verify:

```bash
node --version   # should print v22.x or higher
npm --version
```

<a id="codex-cli-mac"></a>
<details>
<summary><b>Codex CLI</b> — OpenAI's terminal coding agent</summary>

<br>

#### Install

```bash
npm install -g @openai/codex
```

Requires macOS 12+. Verify:

```bash
codex --version
```

To update later:

```bash
npm update -g @openai/codex
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

</details>

<a id="claude-code-mac"></a>
<details>
<summary><b>Claude Code</b> — Anthropic's terminal coding agent</summary>

<br>

#### Install

```bash
npm install -g @anthropic-ai/claude-code
```

Universal binary — works on Apple Silicon and Intel. Verify:

```bash
claude --version
```

To update later:

```bash
npm update -g @anthropic-ai/claude-code
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

</details>

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
