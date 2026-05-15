# Step 1 — Install (detail)

Full install paths, prerequisites, and troubleshooting for getting `codex` and `claude` running on Windows and macOS.

[← Back to README](../README.md)

## Contents

- [Prerequisites](#prerequisites)
- [Codex CLI](#codex-cli)
- [Claude Code](#claude-code)
- [First run / authentication](#first-run--authentication)
- [Troubleshooting](#troubleshooting)

## Prerequisites

Both tools install via npm, so install **Node.js (LTS, v22+)**.

<details>
<summary><b>Windows</b></summary>

<br>

```powershell
winget install -e --id OpenJS.NodeJS.LTS
```

Alternatives: the official installer from [nodejs.org](https://nodejs.org/), or [nvm-windows](https://github.com/coreybutler/nvm-windows) / [fnm](https://github.com/Schniz/fnm) for multiple Node versions side-by-side.

</details>

<details>
<summary><b>Mac</b></summary>

<br>

```bash
brew install node
```

Alternatives: the official installer, or [nvm](https://github.com/nvm-sh/nvm) / [fnm](https://github.com/Schniz/fnm).

</details>

Verify:

```bash
node --version   # v22.x or higher
npm --version
```

## Codex CLI

```bash
npm install -g @openai/codex
```

Verify: `codex --version`. Update later: `npm update -g @openai/codex`.

## Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

Verify: `claude --version`. Update later: `npm update -g @anthropic-ai/claude-code`.

**Windows companion: Git for Windows.** Install [Git for Windows](https://git-scm.com/downloads/win) so Claude Code can use Git Bash as its shell (better than PowerShell for shell-style commands). Without it, Claude Code falls back to PowerShell.

## First run / authentication

```bash
codex   # opens browser → Sign in with ChatGPT
claude  # opens browser → Sign in with Anthropic / Claude account
```

Both store credentials locally; you won't be prompted again. Switch accounts later with `/login` inside the TUI.

For API-key billing instead of subscription auth, set `OPENAI_API_KEY` or `ANTHROPIC_API_KEY` in your environment before launching.

## Troubleshooting

> [!NOTE]
> **Codex CLI exits silently on Windows.** Install the Visual C++ runtime — it's required by the npm build and missing on some bare Windows installs ([issue #20827](https://github.com/openai/codex/issues/20827)):
>
> ```powershell
> winget install -e --id Microsoft.VCRedist.2015+.x64
> ```

> [!NOTE]
> **Claude Code can't find Git Bash on Windows.** Point it at the binary explicitly in `~/.claude/settings.json`:
>
> ```json
> {
>   "env": {
>     "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe"
>   }
> }
> ```

> [!NOTE]
> **`gpt-5.5` not available in Codex CLI.** Run `codex --list-models` to see what your account can select. Default models rotate; use the most capable one your plan supports.

→ Next: [Step 2 — Configure](./configuration.md)
