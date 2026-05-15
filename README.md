# AI Research Setup

A step-by-step guide to setting up AI tools for research, on **Windows** and **macOS**.

Five steps, in order: install → configure → workflow → software → backup. Each step keeps the essentials in this README and links to a sub-page in [docs/](docs/) for the full detail.

> [!TIP]
> **Picking just one?** As of May 2026, GPT models tend to outperform Claude on day-to-day research coding. If your budget only fits one subscription, start with [ChatGPT](https://chatgpt.com/) Plus/Pro and Codex CLI. Claude Code is excellent and the two complement each other well when run together — but a one-tool budget points at OpenAI today. Re-check before committing; rankings flip every few months.

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

This is the one step that requires running commands yourself — you can't ask the AI to install itself. Both tools install via npm, so install Node.js (LTS, v22+) first.

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

Now that you have an AI, ask it to configure itself. Open Codex or Claude Code and paste:

> *"Configure yourself for research work: use the latest model, set thinking effort to xhigh, switch permissions to acceptEdits, and add a sensible statusline. Show me the diff before applying."*

What that actually opts you into:

- **Latest model + max thinking effort.** Codex: `gpt-5.5` + `xhigh`. Claude Code: `claude-opus-4-7` (uses adaptive thinking automatically) or `claude-sonnet-4-6` + `effortLevel: "xhigh"` for explicit control.
- **Skip routine prompts.** `acceptEdits` (Claude Code) / `on-request` (Codex). Once you trust a repo, opt into `auto` per-project — Claude Code's `auto` mode requires a Max+ plan.
- **One keyboard rule for Codex.** **Enter** sends now (or *injects* mid-turn). **Tab** *queues* for the next turn.

→ **Details:** [docs/configuration.md](docs/configuration.md) — full `settings.json` / `config.toml` reference, statusline scripts, permission modes deep-dive.

**Next:** [Step 3 — Workflow](#step-3--typical-research-workflow)

---

## Step 3 — Typical research workflow

> **Goal:** A daily flow that doesn't fight you.

**1. Open the terminal at your project folder.**

- **Windows:** in File Explorer, navigate to the folder. Type `cmd` (or `powershell`, or `wt`) in the address bar and press Enter.
- **Mac:** in Finder, right-click the folder → Services → New Terminal at Folder. Or `cd` from an open terminal.

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

**Snap four windows to quarters.** Windows: `Win+Left/Right` then `Win+Up/Down`. Mac: install [Rectangle](https://rectangleapp.com) for quarter-tiling shortcuts. Or keep all four in one terminal window with internal panes (Windows Terminal: `Alt+Shift+D`; iTerm2: `Cmd+D` / `Cmd+Shift+D`).

**3. Resume yesterday's session** — `claude -c` or `codex resume --last` so context carries over.

**4. Tell the AI to double-check itself.** Append *"double-check your work for accuracy"* to complex requests (derivations, analyses, paper sections). Costs a bit of extra thinking; meaningfully improves accuracy.

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
- **Text → Dropbox + Overleaf.** Edit `.tex` locally in `~/Dropbox/Apps/Overleaf/<project>/` with AI. Overleaf is the canonical render and what coauthors see.

**No Dropbox subscription?** OneDrive works as a substitute for the sync layer.

**Tell your AI the folder convention.** Add this to your project's CLAUDE.md / AGENTS.md:

> Use `~/Dropbox/Code/<project>/` for code. Copy final outputs (tables, figures, csv) to `~/Dropbox/Apps/Overleaf/<project>/` and edit the `.tex` directly there.

For a new project, paste this to the AI:

> *"Create a private GitHub repo for this project, set the remote, and push the initial commit."*

→ **Details:** [docs/github.md](docs/github.md) — `git` + `gh` install/auth (interactive, do this once), what to back up vs. not, the full dual-cloud workflow.

---

## License

[MIT](LICENSE)
