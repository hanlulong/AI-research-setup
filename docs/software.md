# Step 4 — Work with research software (detail)

Get the AI to drive your econ stack.

[← Back to README](../README.md)

## Contents

- [Python, MATLAB, R, Julia](#python-matlab-r-julia)
- [Stata](#stata)
- [Academic writing](#academic-writing)
- [Overleaf](#overleaf)
- [Anything else](#anything-else)
- [General install pattern](#general-install-pattern)

## Python, MATLAB, R, Julia

**No extra setup.** Both Claude Code and Codex CLI can read and write source files for any language and run commands in your project folder. Just ask:

> "Write a Python script that loads `data/main.dta`, runs the regressions in `code/main.do`, and outputs a LaTeX table to `tables/results.tex`."

The AI writes the code, asks permission to run it, and iterates on errors. For larger projects, give it a CLAUDE.md / AGENTS.md with your folder conventions (where data lives, where outputs go, naming, dependency manager).

## Stata

Stata isn't natively callable from a CLI the way Python is, so it needs an MCP bridge.

**Install [stata-mcp](https://github.com/hanlulong/stata-mcp)** — a VS Code / Cursor / Antigravity extension that exposes Stata over MCP:

```bash
code --install-extension DeepEcon.stata-mcp
# or:
cursor --install-extension DeepEcon.stata-mcp
# or:
antigravity --install-extension DeepEcon.stata-mcp
```

The extension runs an MCP server at `http://localhost:4000/mcp-streamable`. Wire it up:

**Claude Code:**

```bash
claude mcp add --transport http stata-mcp http://localhost:4000/mcp-streamable --scope user
```

**Codex CLI** (0.46.0+):

```bash
codex mcp add stata-mcp --url http://localhost:4000/mcp-streamable
```

**Prerequisites:** Stata 17+ installed locally; VS Code / Cursor / Antigravity for the extension; `uv` (auto-installed if missing).

After this, your AI can run `.do` and `.ado` files, view data, and render Stata graphs in real time.

> [!NOTE]
> Initial install takes ~2 minutes (dependency setup). Each parallel session uses 200–300 MB RAM; mind Stata's concurrent-instance license limits. In Cursor / Antigravity, the extension's toolbar buttons are hidden by default — enable via "Configure Icon Visibility."

Full docs: https://github.com/hanlulong/stata-mcp

## Academic writing

[econ-writing-skill](https://github.com/hanlulong/econ-writing-skill) is a Claude Code / Codex skill synthesizing 50+ writing guides from Cochrane, McCloskey, Shapiro, Head, Bellemare, Goldin, Kremer, and others.

**Install (one-liner):**

```bash
curl -fsSL https://raw.githubusercontent.com/hanlulong/econ-writing-skill/main/install.sh | bash
```

Flags: `--local` (current project only), `--claude` / `--codex` (target a specific client).

**Restart your AI session**, then invoke:

```
/econ-write write the introduction for my paper on minimum-wage incidence
/econ-write rewrite this paragraph in McCloskey's voice
/econ-write audit my full paper and score it
/econ-write draft a referee response to this comment
```

Works for drafts, audits, referee responses, JMPs, grant proposals, conference talks.

## Overleaf

Two patterns, depending on your Overleaf plan.

### Pattern 1 — Dropbox sync (Overleaf Premium)

Overleaf's Dropbox integration syncs your `<project>` folder to `~/Dropbox/Apps/Overleaf/<project>/`. Edit `.tex` files locally with AI; Overleaf pulls changes automatically.

This is the cleanest setup if you have Overleaf Premium.

### Pattern 2 — overleaf-sync-now (for the sync lag)

Dropbox-to-Overleaf sync runs every ~10–20 minutes. If you edit on Overleaf's web UI and then ask your AI to modify the same file locally, the AI may be working on a stale copy.

[overleaf-sync-now](https://github.com/hanlulong/overleaf-sync-now) is a CLI + Claude Code PreToolUse hook that pulls fresh Overleaf web edits before the AI reads/edits a `.tex` / `.bib` / `.cls` / `.sty` / `.bst` file.

**Install via your AI (easiest):**

Open Claude Code or Codex and paste:

> Install overleaf-sync-now from https://github.com/hanlulong/overleaf-sync-now using `uv tool install`, then run `overleaf-sync-now install`.

**Manual install (Mac/Linux):**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
export PATH="$HOME/.local/bin:$PATH"
uv tool install --from git+https://github.com/hanlulong/overleaf-sync-now overleaf-sync-now
overleaf-sync-now install
```

**Manual install (Windows PowerShell):**

```powershell
irm https://astral.sh/uv/install.ps1 | iex
$env:PATH = "$env:USERPROFILE\.local\bin;$env:PATH"
uv tool install --from git+https://github.com/hanlulong/overleaf-sync-now overleaf-sync-now
overleaf-sync-now install
```

Restart your AI after install (`/exit` then `claude`). Verify with `overleaf-sync-now status`.

> [!NOTE]
> On Windows with Chrome 130+, run `overleaf-sync-now login` once — Chrome's App-Bound Encryption blocks silent cookie extraction.

## Anything else

[awesome-ai-for-economists](https://github.com/hanlulong/awesome-ai-for-economists) is a curated directory of AI tools for economists. Categories indexed:

- **MCP servers for economic data** — FRED, BLS, Census, Eurostat, IMF, OECD, World Bank, Alpha Vantage, Nasdaq Data Link, OpenEcon, …
- **Coding tools and agent frameworks** — Stata-MCP, Aider, Cline, Jupyter AI, Marimo, Positron, AutoGen, LangGraph, DSPy, Pydantic AI, …
- **Causal inference** — DoubleML, DoWhy, EconML, grf, CausalPy, CausalML, CausalAgent, …
- **Forecasting / nowcasting** — Chronos, TimeGPT, TimesFM, Lag-Llama, neuralforecast
- **Simulation, game theory, DSGE** — Nashpy, pygambit, Axelrod, gEconpy, MacroModelling.jl, optimagic
- **Literature review** — Elicit, Consensus, SciSpace, Connected Papers, PaperQA2, STORM, Undermind, OpenScholar
- **Academic writing** — Econ Writing Skill, Overleaf AI Assist, Paperpal, Refine.ink, Thesify, Underleaf, Zotero PapersGPT
- **Document processing / OCR** — Docling, Marker, Mathpix, MinerU, OlmOCR
- **NLP for economics**, **policy / labor / alternative data**, **finance**, **data collection**

Browse the list and pick what's useful for your project.

## General install pattern

For any new tool you find online (in the directory above, or anywhere):

> Open Claude Code or Codex CLI, paste the GitHub repo URL, and ask: **"install this."**

The AI reads the README and runs the install commands. Faster than copy-pasting them yourself, handles platform variants automatically, and asks for permission before each system change.

→ Next: [Step 5 — Backup & cloud](./github.md)
