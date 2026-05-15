# AI Research Setup

A step-by-step guide to setting up AI tools for research, on **Windows** and **macOS**.

Five steps, in order: install → configure → workflow → software → backup. Each step keeps the essentials in this README and links to a sub-page in [docs/](docs/) for the full detail.

> [!TIP]
> **Picking just one?** As of May 2026, GPT models tend to outperform Claude on day-to-day research coding. If your budget only fits one subscription, start with [ChatGPT](https://chatgpt.com/) Plus/Pro and Codex CLI. Claude Code is excellent and the two complement each other well when run together — but a one-tool budget points at OpenAI today. Re-check before committing; rankings flip every few months.

## Contents

1. [Install](#step-1--install)
2. [Configure & use effectively](#step-2--configure--use-effectively)
3. [Typical research workflow](#step-3--typical-research-workflow)
4. [Work with research software](#step-4--work-with-research-software)
5. [Backup & cloud](#step-5--backup--cloud)

---

## Step 1 — Install

> **Goal:** `codex` and `claude` running in your terminal.

Both install via npm, so install Node.js (LTS, v22+) first.

**Windows** (PowerShell):

```powershell
winget install -e --id OpenJS.NodeJS.LTS
npm install -g @openai/codex
npm install -g @anthropic-ai/claude-code
```

**macOS:**

```bash
brew install node
npm install -g @openai/codex
npm install -g @anthropic-ai/claude-code
```

Verify with `codex --version` and `claude --version`. First-time launch of each (`codex` or `claude`) opens a browser to authenticate against your ChatGPT or Claude account.

→ **Details:** [docs/install.md](docs/install.md) — alternative install paths, prerequisites, troubleshooting.

**Next:** [Step 2 — Configure](#step-2--configure--use-effectively)

---

## Step 2 — Configure & use effectively

> **Goal:** Both tools running at maximum capability.

Defaults are tuned for caution. Three opt-ins make them research-ready:

**1. Latest model + max thinking effort.** As of May 2026:

| | Codex CLI (`~/.codex/config.toml`) | Claude Code (`~/.claude/settings.json`) |
|---|---|---|
| Model | `model = "gpt-5.5"` | `"model": "claude-opus-4-7"` |
| Thinking | `model_reasoning_effort = "xhigh"` | `"effortLevel": "xhigh"`, `"alwaysThinkingEnabled": true` |

**2. Skip routine prompts.** Set `permissions.defaultMode = "acceptEdits"` in Claude Code; set `approval_policy = "on-request"` in Codex. Once you trust a repo, opt into `auto` mode per-project.

**3. One keyboard rule for Codex.** **Enter** sends immediately — and if Codex is mid-turn, *injects* your message into the running turn. **Tab** *queues* your message for the next turn. Use Tab to add context without interrupting.

→ **Details:** [docs/configuration.md](docs/configuration.md) — full `settings.json` / `config.toml`, statusline, permission modes, more shortcuts.

**Next:** [Step 3 — Workflow](#step-3--typical-research-workflow)

---

## Step 3 — Typical research workflow

> **Goal:** A daily flow that doesn't fight you.

**1. Open the terminal at your project folder.**

- **Windows:** in File Explorer, navigate to the folder. Type `cmd` in the address bar and press Enter — opens CMD in that folder. (`powershell` and `wt` also work.)
- **Mac:** in Finder, right-click the folder → Services → New Terminal at Folder. Or `cd` from an open terminal.

**2. Run a 4-pane window** so you see editor, AI, output, and reference at once:

```
┌──────────────────┬──────────────────┐
│  Editor          │  Browser /       │
│  (VS Code,       │  Notes /         │
│   Cursor)        │  Reference docs  │
├──────────────────┼──────────────────┤
│  Codex CLI       │  Claude Code     │
└──────────────────┴──────────────────┘
```

iTerm2 (Mac): `Cmd+D` / `Cmd+Shift+D` to split. Windows Terminal: `Alt+Shift+D`.

**3. Resume yesterday's session** — `claude -c` or `codex --resume` so context carries over.

→ **Details:** [docs/workflow.md](docs/workflow.md) — pane shortcuts, parallel agents, when to use plan vs auto, daily flow.

**Next:** [Step 4 — Software](#step-4--work-with-research-software)

---

## Step 4 — Work with research software

> **Goal:** AI that can drive your econ stack.

| Software | Setup |
|---|---|
| **Python, MATLAB, R, Julia** | Native — just ask the AI. No extra setup. |
| **Stata** | Install [stata-mcp](https://github.com/hanlulong/stata-mcp); wire it to your AI as an MCP server. |
| **Academic writing** | Install [econ-writing-skill](https://github.com/hanlulong/econ-writing-skill) — `/econ-write` skill grounded in 50+ econ writing guides. |
| **Overleaf** | Use Overleaf's Dropbox sync; edit `.tex` files locally. Install [overleaf-sync-now](https://github.com/hanlulong/overleaf-sync-now) if the sync lag bites. |
| **Anything else** | [awesome-ai-for-economists](https://github.com/hanlulong/awesome-ai-for-economists) — curated directory of MCP servers, skills, models, and tools. |

> [!TIP]
> **Install pattern for any new tool:** open Claude Code or Codex CLI, paste the GitHub repo URL, and ask "install this." The AI reads the README and runs the commands.

→ **Details:** [docs/software.md](docs/software.md) — per-tool install commands, prereqs, MCP wiring.

**Next:** [Step 5 — Backup](#step-5--backup--cloud)

---

## Step 5 — Backup & cloud

> **Goal:** Never lose work. Code versioned, text canonical, both synced.

**Recommended dual-cloud setup:**

- **Code → Dropbox + GitHub.** Dropbox is your live working folder (auto-syncs across machines). GitHub is your versioned backup (history, branches, sharing). Push regularly.
- **Text → Dropbox + Overleaf.** Edit `.tex` locally in `~/Dropbox/Apps/Overleaf/<project>/` with AI. Overleaf is the canonical render and what coauthors see.

**No Dropbox subscription?** OneDrive works as a substitute for the sync layer.

**Tell your AI the folder convention.** Add this to your project's CLAUDE.md / AGENTS.md:

> Use `~/Dropbox/Code/<project>/` for code. Copy final outputs (tables, figures, csv) to `~/Dropbox/Apps/Overleaf/<project>/` and edit the `.tex` directly there.

**Create a GitHub repo for the project:**

```bash
cd ~/Dropbox/Code/<project>
gh repo create --source=. --remote=origin --push --private  # or --public
```

→ **Details:** [docs/github.md](docs/github.md) — `git` + `gh` install/auth, what to back up, the full dual-cloud workflow.

---

## License

[MIT](LICENSE)
