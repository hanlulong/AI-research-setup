# Step 4 — Work with research software (detail)

Get the AI to drive your econ stack.

[← Back to README](../README.md)

> [!IMPORTANT]
> **Easy path: ask the AI.** For each tool below, the recommended path is to paste the "Ask the AI" prompt into Codex or Claude Code. The AI reads the tool's README and runs the commands. Manual command paths are tucked into "Or manually" blocks for reference.

## Contents

- [Python, MATLAB, R, Julia](#python-matlab-r-julia)
- [Stata](#stata)
- [Academic writing](#academic-writing)
- [Overleaf](#overleaf)
- [Anything else](#anything-else)

## Python, MATLAB, R, Julia

**No extra setup.** Both Claude Code and Codex CLI can read and write source files for any language and run commands in your project folder. Just ask:

> *"Write a Python script that loads `data/main.dta`, runs the regressions in `code/main.do`, and outputs a LaTeX table to `tables/results.tex`."*

The AI writes the code, runs it (pausing to confirm anything risky), and iterates on errors. For larger projects, give it a CLAUDE.md / AGENTS.md with your folder conventions (where data lives, where outputs go, naming, dependency manager).

### Optional: polished MATLAB plots

For publication-quality MATLAB figures, install [matlab-plot-skill](https://github.com/hanlulong/matlab-plot-skill) — a Claude Code / Codex skill that refactors messy `.m` plotting code, renders the figure, and iterates to a clean PDF.

**Ask the AI:**

> *"Install matlab-plot-skill from https://github.com/hanlulong/matlab-plot-skill."*

Invoke it with `/matlab-plot-skill` in Claude Code or `$matlab-plot-skill` in Codex:

```
# Claude Code
/matlab-plot-skill refactor Plots/generate_my_figure.m into a publication-quality PDF figure.
# Codex
Use $matlab-plot-skill to refactor Plots/generate_my_figure.m into a publication-quality PDF figure.
```

**Prerequisites:** MATLAB R2021a or newer, with `matlab` on your PATH (on macOS it often isn't after install; see the [skill's README](https://github.com/hanlulong/matlab-plot-skill#prerequisites)). Installer: `install.sh` (macOS/Linux) or `install.ps1` (Windows); `uv` and `pyyaml` are only needed to validate the repo.

## Stata

Stata needs an MCP bridge for AI tools to drive it interactively. [stata-mcp](https://github.com/hanlulong/stata-mcp) is a VS Code / Cursor / Antigravity extension that exposes Stata over MCP.

**Ask the AI:**

> *"Install stata-mcp from https://github.com/hanlulong/stata-mcp and wire it up as an MCP server so you can run .do files."*

The AI installs the extension, starts the MCP server, and runs the right `mcp add` command for whichever tool you're using.

**Prerequisites:** Stata 17+ installed locally; VS Code / Cursor / Antigravity for the extension.

<details>
<summary><b>Or manually</b></summary>

<br>

Install the extension:

```bash
code --install-extension DeepEcon.stata-mcp
# or: cursor --install-extension DeepEcon.stata-mcp
# or: antigravity --install-extension DeepEcon.stata-mcp
```

Then open your IDE (VS Code / Cursor / Antigravity) so the extension activates and starts its MCP server on port 4000 — the `mcp add` commands below point at it.

Wire to Claude Code:

```bash
claude mcp add --transport http stata-mcp http://localhost:4000/mcp-streamable --scope user
```

Or to Codex CLI (requires 0.46.0 or newer for `codex mcp add`):

```bash
codex mcp add stata-mcp --url http://localhost:4000/mcp-streamable
```

Restart Claude Code or Codex after adding it, since MCP tool lists don't refresh mid-session. Health check: `curl -s http://localhost:4000/health`.

</details>

> [!NOTE]
> Initial install takes ~2 minutes. Each parallel session uses 200–300 MB RAM; mind Stata's concurrent-instance license limits. In Cursor / Antigravity, the extension's toolbar buttons are hidden by default — enable via "Configure Icon Visibility."

Full docs: https://github.com/hanlulong/stata-mcp

## Academic writing

[econ-writing-skill](https://github.com/hanlulong/econ-writing-skill) is a Claude Code / Codex skill synthesizing 50+ writing guides from Cochrane, McCloskey, Shapiro, Head, Bellemare, Goldin, Kremer, and others.

**Ask the AI:**

> *"Install the econ-writing-skill from https://github.com/hanlulong/econ-writing-skill."*

Restart your AI session after install, then invoke it with `/econ-write` in Claude Code or `$econ-write` in Codex:

```
/econ-write write the introduction for my paper on minimum-wage incidence
/econ-write rewrite this paragraph in McCloskey's voice
/econ-write audit my full paper and score it
/econ-write draft a referee response to this comment
```

Works for drafts, audits, referee responses, JMPs, grant proposals, conference talks.

<details>
<summary><b>Or manually</b></summary>

<br>

```bash
# macOS / Linux: pick exactly one of --claude, --codex, --all
curl -fsSL https://raw.githubusercontent.com/hanlulong/econ-writing-skill/main/scripts/install.sh | bash -s -- --global --claude
```

Exactly one of `--claude`, `--codex` or `--all` is required; `--local` installs for one project only. On Windows, or anywhere with Node.js and `npx`: `npx skills add hanlulong/econ-writing-skill`. Current options: [INSTALL.md](https://github.com/hanlulong/econ-writing-skill/blob/main/INSTALL.md).

</details>

## Overleaf

Two patterns, depending on your Overleaf plan.

### Pattern 1 — Dropbox sync (Overleaf Premium)

Overleaf's Dropbox integration syncs your `<project>` folder to `~/Dropbox/Apps/Overleaf/<project>/`. Edit `.tex` files locally with AI; Overleaf pulls changes automatically.

This is the cleanest setup if you have Overleaf Premium. No install required.

### Pattern 2 — overleaf-sync-now (for the sync lag)

Overleaf's Dropbox sync can take ~10–20 minutes to bring edits made on overleaf.com down to your local Dropbox folder (overleaf-sync-now's measurement; Overleaf doesn't document an interval). If you edit on Overleaf's web UI and then ask your AI to modify the same file locally, the AI may be working on a stale copy.

[overleaf-sync-now](https://github.com/hanlulong/overleaf-sync-now) is an agent skill plus a CLI and PreToolUse hooks for Claude Code and Codex CLI that pull fresh Overleaf web edits before the AI reads/edits a `.tex` / `.bib` / `.cls` / `.sty` / `.bst` file.

**Ask the AI:**

> *"Install overleaf-sync-now from https://github.com/hanlulong/overleaf-sync-now using `uv tool install` at its latest release tag, then run `overleaf-sync-now install`."*

Restart your AI after install (`/exit` then relaunch). Verify with `overleaf-sync-now status`.

In Codex CLI, also run `/hooks` once and trust the overleaf-sync-now hook; Codex skips new hooks until you do.

<details>
<summary><b>Or manually</b></summary>

<br>

**Mac/Linux:**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
export PATH="$HOME/.local/bin:$PATH"
uv tool install --from git+https://github.com/hanlulong/overleaf-sync-now@v0.5.0 overleaf-sync-now
overleaf-sync-now install
```

**Windows PowerShell:**

```powershell
irm https://astral.sh/uv/install.ps1 | iex
$env:PATH = "$env:USERPROFILE\.local\bin;$env:PATH"
uv tool install --from git+https://github.com/hanlulong/overleaf-sync-now@v0.5.0 overleaf-sync-now
overleaf-sync-now install
```

(Swap `v0.5.0` for the latest tag in the [overleaf-sync-now README](https://github.com/hanlulong/overleaf-sync-now).)

</details>

> [!NOTE]
> If `overleaf-sync-now install` ends with **AUTH NOT YET CAPTURED** (any OS), run `overleaf-sync-now login`. This is also the fix on Windows with Chrome 127+ (App-Bound Encryption) and for Safari on macOS.

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

Browse the list, find a tool you want, and:

> *"Install <tool name> from <repo URL>."*

The AI reads the README and runs the commands.

→ Next: [Step 5 — Backup & cloud](./github.md)
