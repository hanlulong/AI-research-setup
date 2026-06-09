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

None for the native installers below — they bundle their own runtime, so **Node.js is not required** to install or run either tool.

You only need **Node.js (LTS, v22+)** if you take the npm install path instead, or plan to run Node-based MCP servers (e.g. the `context7` example in [configuration](./configuration.md#configtoml), which launches via `npx`). To install it then: Windows `winget install -e --id OpenJS.NodeJS.LTS`; macOS `brew install node` (alternatives: [nodejs.org](https://nodejs.org/), or [nvm](https://github.com/nvm-sh/nvm) / [fnm](https://github.com/Schniz/fnm) for multiple versions side-by-side).

## Codex CLI

Official standalone installer — no Node required ([docs](https://developers.openai.com/codex/cli)):

<details>
<summary><b>Windows</b> (PowerShell)</summary>

<br>

```powershell
irm https://chatgpt.com/codex/install.ps1 | iex
```

</details>

<details>
<summary><b>Mac</b></summary>

<br>

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

</details>

Alternatives: `brew install --cask codex` (macOS), or the npm package `npm install -g @openai/codex` (needs Node).

Verify: `codex --version` (expect 0.46.0 or newer — needed for the `codex mcp add` step later). The standalone installer self-updates; on npm, update with `npm install -g @openai/codex@latest`.

## Claude Code

Official native installer — recommended, no Node required, auto-updates in the background ([docs](https://code.claude.com/docs/en/setup)):

<details>
<summary><b>Windows</b> (PowerShell)</summary>

<br>

```powershell
irm https://claude.ai/install.ps1 | iex
```

</details>

<details>
<summary><b>Mac</b></summary>

<br>

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

</details>

Alternatives: `brew install --cask claude-code` (macOS), `winget install Anthropic.ClaudeCode` (Windows), or the npm package `npm install -g @anthropic-ai/claude-code` (needs Node 18+).

Verify: `claude --version` (expect 2.1.83 or newer — `auto` permission mode needs it). Native installs auto-update; on npm, update with `npm install -g @anthropic-ai/claude-code@latest`.

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
> **Codex CLI exits silently on Windows.** Install the Visual C++ runtime — Codex needs it and it's missing on some bare Windows installs ([issue #20827](https://github.com/openai/codex/issues/20827)):
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
> **Picking a Codex model.** `gpt-5.5` is the default; `gpt-5.5-pro` needs a higher-tier plan. Plan tier mainly affects usage caps. Defaults rotate — open the `/model` picker inside Codex to see what your account can actually select.

→ Next: [Step 2 — Configure](./configuration.md)
