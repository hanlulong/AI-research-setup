# AI Research Setup

A step-by-step guide to setting up AI tools for research, on **Windows** and **macOS**.

Five steps, in order: install → configure → workflow → software → backup. Each step keeps the essentials in this README and links to a sub-page in [docs/](docs/) for the full detail.

> [!TIP]
> **Picking just one?** As of June 2026, GPT models still edge out Claude on day-to-day research coding. If your budget only fits one subscription, start with ChatGPT Plus/Pro and [Codex CLI](https://developers.openai.com/codex/cli). [Claude Code](https://claude.com/claude-code) is excellent and the two complement each other well when run together — but a one-tool budget still points at OpenAI today.

> [!IMPORTANT]
> **After Step 1, ask the AI.** Once `claude` or `codex` is running, paste your intent (e.g. *"install stata-mcp and wire it up"*) and let the AI handle the commands. Sub-pages still document the raw commands, but the recommended path is to use the AI as your installer and configurator.

## Contents

1. [Install](#step-1--install)
2. [Configure & use effectively](#step-2--configure--use-effectively)
3. [Typical research workflow](#step-3--typical-research-workflow)
4. [Work with research software](#step-4--work-with-research-software)
5. [Backup & cloud](#step-5--backup--cloud)

---

## Step 1 — Install

> **Goal:** `codex` and `claude` running in your terminal.

This step requires running commands yourself — the AI can't install itself yet. The official installers below are self-contained — no Node.js required.

**Windows** (PowerShell):

```powershell
irm https://chatgpt.com/codex/install.ps1 | iex   # Codex CLI
irm https://claude.ai/install.ps1 | iex           # Claude Code
```

**macOS:**

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh     # Codex CLI
curl -fsSL https://claude.ai/install.sh | bash           # Claude Code
```

Verify with `codex --version` and `claude --version`. The first run of either tool opens a browser to authenticate against your ChatGPT or Claude account. If a command isn't found or Codex exits with no output, see [troubleshooting](docs/install.md#troubleshooting) (older Codex npm releases could need the Visual C++ runtime on bare Windows).

**Windows:** also install [Git for Windows](https://git-scm.com/downloads/win) (`winget install -e --id Git.Git`) so Claude Code can use Git Bash.

→ **Details:** [docs/install.md](docs/install.md) — Homebrew/npm alternatives, prerequisites, troubleshooting.

**Next:** [Step 2 — Configure](#step-2--configure--use-effectively)

---

## Step 2 — Configure & use effectively

> **Goal:** Both tools running at maximum capability.

Ask Codex or Claude Code to configure itself:

> *"Configure yourself for research work: use the latest model, set reasoning effort to xhigh, keep a sensible permission mode, and add a sensible statusline. Show me the diff before applying."*

What that actually opts you into:

- **Model + deep reasoning.**
  - Codex: `gpt-6.1-sol` + `xhigh` (`gpt-6-astra` is the most capable, at lower usage limits).
  - Claude Code: `claude-opus-5-5` (alias `opus`); set `xhigh` with `/effort xhigh`, typed once per model and saved per model. Opus 5.5 defaults to `medium` and ignores a top-level `effortLevel` in user settings; `xhigh` uses noticeably more of your plan; `max` is the deepest, session-only level. For the longest, hardest tasks, `/model fable` (may bill to usage credits).
- **Maximum thoroughness on demand.** **Ultracode** is automatic multi-agent workflows at your session's effort (pair it with `xhigh`).
  - Turn it on with `/effort ultracode` per session (`/effort ultracode off` to stop), or make it your default with `"ultracode": true` in `~/.claude/settings.json` (works on Windows and macOS).
  - It needs Dynamic workflows (all paid plans; on Pro, turn them on in `/config`) and uses noticeably more usage. Reach for it on complex or large projects.
  - For quick iteration, use a lower effort level; `/fast` is the same Opus ~2.5× faster but bills to usage credits on subscriptions.
- **Auto mode for execution.**
  - Claude Code (v2.1.283+) starts interactive sessions in `auto` when your model supports it (otherwise Manual): lower-risk tool calls run automatically after a risk and prompt-injection check, and the rest are blocked. `claude -p` scripts start in Manual. Pin it with `"defaultMode": "auto"` in user settings if you like.
  - Codex: `on-request` (or `never` per-project).
  - Drop to `plan` (read-only) or `acceptEdits` when you want a tighter leash.
- **One keyboard rule for Codex.** **Enter** sends now (or *injects* mid-turn). **Tab** *queues* for the next turn.

→ **Details:** [docs/configuration.md](docs/configuration.md) — full `settings.json` / `config.toml` reference, statusline scripts, permission modes deep-dive.

**Next:** [Step 3 — Workflow](#step-3--typical-research-workflow)

---

## Step 3 — Typical research workflow

> **Goal:** A daily flow that doesn't fight you.

**1. Open the terminal at your project folder.**

- **Windows:** in File Explorer, navigate to the folder. Type `cmd` (or `powershell`, or `wt`) in the address bar and press Enter.
- **Mac:** in Finder, right-click the folder → Services → New Terminal at Folder (if it's missing, enable it in System Settings → Keyboard → Keyboard Shortcuts → Services). Or `cd` from an open terminal.

**2. Run a 4-pane window with an AI agent in each pane** — each working on its own project or task in parallel:

```
┌──────────────────┬──────────────────┐
│  AI agent 1      │  AI agent 2      │
│  Project A       │  Project B       │
├──────────────────┼──────────────────┤
│  AI agent 3      │  AI agent 4      │
│  Project C       │  Task X          │
└──────────────────┴──────────────────┘
```

**Snap four windows to quarters.** Windows: `Win+Left/Right` then `Win+Up/Down`. Mac (15+): Window → Move & Resize → Top Left/Top Right/Bottom Left/Bottom Right; for keyboard shortcuts, install [Rectangle](https://rectangleapp.com). Or keep all four in one terminal window with internal panes (Windows Terminal: `Alt+Shift+D`; iTerm2: `Cmd+D` / `Cmd+Shift+D`).

**3. Resume yesterday's session** — `claude -c` or `codex resume --last` so context carries over.

> [!TIP]
> **Ask the AI to double-check after it finishes.** When the AI completes any complex task, send a brief follow-up: *"Double-check everything is correct. Zero errors. Improve. Make everything professional."* It re-examines its own work — catches mistakes, polishes weak spots. One of the cheapest accuracy boosts available. For code changes specifically, run `/code-review` in Claude Code or `/review` in Codex (`/code-review ultra` launches a deep multi-agent cloud review of the branch or a PR; user-triggered, billed).

→ **Details:** [docs/workflow.md](docs/workflow.md) — pane shortcuts, parallel agents, daily flow.

**Next:** [Step 4 — Software](#step-4--work-with-research-software)

---

## Step 4 — Work with research software

> **Goal:** AI that can drive your econ stack.

**General pattern:** paste a tool's GitHub URL into Claude Code or Codex and ask **"install this."** For research-specific tools:

| Software | What to paste |
|---|---|
| **Python, MATLAB, R, Julia** | Native — no install needed. Just describe the task. |
| **Stata** | *"Install stata-mcp from https://github.com/hanlulong/stata-mcp and wire it up as an MCP server so you can run .do files."* |
| **Academic writing** | *"Install the econ-writing-skill from https://github.com/hanlulong/econ-writing-skill."* |
| **Overleaf** | Use Overleaf's Dropbox sync. If the 10–20-minute sync lag bites: *"Install overleaf-sync-now from https://github.com/hanlulong/overleaf-sync-now."* |
| **Anything else** | Browse [awesome-ai-for-economists](https://github.com/hanlulong/awesome-ai-for-economists), pick a URL, ask the AI to install. |

→ **Details:** [docs/software.md](docs/software.md) — what each tool does, prereqs, the raw install commands.

**Next:** [Step 5 — Backup](#step-5--backup--cloud)

---

## Step 5 — Backup & cloud

> **Goal:** Never lose work. Code versioned, text canonical, both synced.

**Recommended dual-cloud setup:**

- **Code → Dropbox + GitHub.** Dropbox is your live working folder (auto-syncs across machines). GitHub is your versioned backup (history, branches, sharing). Push regularly.
- **Text → Dropbox + Overleaf.** Edit `.tex` files locally in `~/Dropbox/Apps/Overleaf/<project>/`. Overleaf is the canonical render and what coauthors see.

**No Dropbox subscription?** OneDrive works as a substitute for the sync layer.

**Tell your AI the folder convention.** Add this to your project's CLAUDE.md / AGENTS.md:

> Use `~/Dropbox/Code/<project>/` for code. Copy final outputs (tables, figures, CSV files) to `~/Dropbox/Apps/Overleaf/<project>/` and edit the `.tex` files directly there.

For a new project, ask the AI:

> *"Create a private GitHub repo for this project, set the remote, and push the initial commit."*

→ **Details:** [docs/github.md](docs/github.md) — `git` + `gh` install/auth (interactive, do this once), what to back up vs. not, the full dual-cloud workflow.

---

## License

[MIT](LICENSE)
