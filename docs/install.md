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

**macOS:** the `brew` commands in this guide need [Homebrew](https://brew.sh); `git` comes with Apple's Command Line Tools (`xcode-select --install`).

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

Verify: `codex --version` (expect 0.134.0 or newer: profile files like `fast.config.toml` need it). Update with `codex update` (or re-run the installer); on npm, update with `npm install -g @openai/codex@latest`.

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

Alternatives: `brew install --cask claude-code` (macOS), `winget install Anthropic.ClaudeCode` (Windows), or the npm package `npm install -g @anthropic-ai/claude-code` (needs Node 22+). Homebrew and WinGet installs don't auto-update: run `brew upgrade claude-code` or `winget upgrade Anthropic.ClaudeCode`.

Verify: `claude --version` (expect 2.1.284 or newer: Opus 5.5, `auto` as the starting mode and the current ultracode toggle need it). Native installs auto-update; `claude update` updates now, and `claude doctor` prints read-only diagnostics. On npm, update with `npm install -g @anthropic-ai/claude-code@latest`.

**Windows companion: Git for Windows.** Install [Git for Windows](https://git-scm.com/downloads/win) so Claude Code can use Git Bash as its shell (better than PowerShell for shell-style commands). Without it, Claude Code falls back to PowerShell.

## First run / authentication

```bash
codex   # opens browser → Sign in with ChatGPT
claude  # opens browser → Sign in with Anthropic / Claude account
```

Both store credentials locally; you won't be prompted again. Switch accounts later with `/login` inside Claude Code; in Codex, run `/logout`, then `codex login`.

For API-key billing instead of subscription auth: Claude Code reads `ANTHROPIC_API_KEY` from your environment. Codex needs a one-time login with the key; on macOS run `printenv OPENAI_API_KEY | codex login --with-api-key`, and on Windows see the [Codex auth docs](https://learn.chatgpt.com/docs/auth) or ask the AI. (`CODEX_API_KEY` is only for non-interactive runs such as `codex exec`.)

## Troubleshooting

> [!NOTE]
> **Codex CLI exits silently on Windows.** Older Codex releases installed via npm could exit silently on bare Windows without the Visual C++ runtime ([issue #20827](https://github.com/openai/codex/issues/20827), fixed by [openai/codex#22905](https://github.com/openai/codex/pull/22905)). Update Codex first; if it still exits silently, install the runtime:
>
> ```powershell
> winget install -e --id Microsoft.VCRedist.2015+.x64
> ```

> [!NOTE]
> **PowerShell blocks the installer script?** Use the vendor form `powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"` (same pattern for the other `irm` installers).

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
> **Picking a Codex model.** GPT-5.5 was retired from Codex for ChatGPT sign-in on 2026-10-14 (API-key sessions keep it). Use `gpt-6.1-sol`, or `gpt-6-astra` for the hardest work at lower usage limits. On Free and Go, OpenAI lists Codex (GPT-6 Luna) only in the ChatGPT desktop app, subject to rollout. Plan tier mainly affects usage caps; open the `/model` picker inside Codex to see what your account can select.

→ Next: [Step 2 — Configure](./configuration.md)
