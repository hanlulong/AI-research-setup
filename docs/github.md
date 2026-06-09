# Step 5 — Backup & cloud (detail)

Two parts: (a) install and authenticate the GitHub CLI, then (b) the full dual-cloud backup pattern — Dropbox + GitHub for code, Dropbox + Overleaf for text.

[← Back to README](../README.md)

> [!IMPORTANT]
> **Most of this is AI-doable.** Once Codex or Claude is running, ask it to set your git identity, apply sensible defaults, and create/push repos — Claude Code's `auto` mode (once you enable it) runs lower-risk commands like these for you and pauses on anything risky. You only need to run the `git`/`gh` install and the interactive `gh auth login` (browser flow) yourself.

## Contents

- [Install git and gh](#install-git-and-gh)
- [Configure](#configure)
- [Backup pattern](#backup-pattern)
- [Create a project repo](#create-a-project-repo)
- [What to back up](#what-to-back-up)
- [Installing more tools](#installing-more-tools)

## Install git and gh

Both AI tools assume you have **git** and the **GitHub CLI** (`gh`) installed and authenticated. `gh auth login` is the easiest path — it handles HTTPS tokens *and* SSH keys in one interactive flow, and gives Claude Code / Codex CLI a working `gh` for PR and issue commands out of the box.

<details>
<summary><b>Windows</b></summary>

<br>

```powershell
winget install -e --id Git.Git
winget install -e --id GitHub.cli
```

If you already installed [Git for Windows](https://git-scm.com/downloads/win) as the Claude Code companion, the first command is a no-op.

</details>

<details>
<summary><b>Mac</b></summary>

<br>

```bash
brew install git gh
```

</details>

Verify both are on your PATH:

```bash
git --version
gh --version
```

## Configure

Two things: identity + defaults (AI can do these), then `gh auth login` (you must run yourself — browser flow).

**Ask the AI:**

> *"Set my git global identity to 'Your Name' <you@example.com> and apply sensible defaults: init.defaultBranch=main, pull.rebase=false, push.autoSetupRemote=true."*

<details>
<summary><b>Or manually</b></summary>

<br>

```bash
git config --global user.name  "Your Name"
git config --global user.email you@example.com
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global push.autoSetupRemote true
```

</details>

Then **authenticate** in your terminal (browser opens):

```bash
gh auth login
```

It asks four things:

1. **GitHub.com or GitHub Enterprise?** → GitHub.com
2. **Preferred protocol for git operations?** → SSH (`gh` will offer to generate and upload a key for you)
3. **Upload your SSH public key to your GitHub account?** → Yes
4. **How would you like to authenticate?** → Login with a web browser

Verify:

```bash
gh auth status                # should show "Logged in to github.com as <you>"
ssh -T git@github.com         # should say "Hi <you>! You've successfully authenticated..."
gh repo list                  # lists your repos — confirms API access
```

After this, `git clone git@github.com:...`, `gh repo create`, `gh pr create`, and AI-tool GitHub integrations all work without further setup.

## Backup pattern

**Recommended dual-cloud setup:**

- **Code → Dropbox + GitHub.** Dropbox = live working folder (instant sync across machines, instant recovery). GitHub = versioned backup (history, branches, sharing, code review).
- **Text → Dropbox + Overleaf.** Dropbox = local edit space for AI. Overleaf = canonical render and what coauthors see.

```
                ┌─────────────────┐
                │   You + AI      │
                │   (local edits) │
                └────────┬────────┘
                         │
              ┌──────────┴──────────┐
        ┌─────▼─────┐         ┌─────▼─────┐
        │  Dropbox  │         │  Dropbox  │
        │  /Code/   │         │  /Apps/   │
        │           │         │  Overleaf │
        └─────┬─────┘         └─────┬─────┘
              │                     │
        ┌─────▼─────┐         ┌─────▼─────┐
        │  GitHub   │         │ Overleaf  │
        │ versioned │         │ canonical │
        └───────────┘         └───────────┘
```

**No Dropbox subscription?** OneDrive works as a drop-in substitute for the sync layer.

**Tell your AI the folder convention.** Add this to your project's CLAUDE.md / AGENTS.md so the AI does the right thing automatically:

> Use `~/Dropbox/Code/<project>/` for code. Copy final outputs (tables, figures, CSV files) to `~/Dropbox/Apps/Overleaf/<project>/` and edit the `.tex` files directly there.

## Create a project repo

**Ask the AI from inside the project folder:**

> *"Create a private GitHub repo for this project, set the remote, and push the initial commit."*

(Use "public" instead if the project should be open.) The AI runs `gh repo create`, scaffolds a `.gitignore` if you need one, and pushes.

<details>
<summary><b>Or manually</b></summary>

<br>

```bash
cd ~/Dropbox/Code/<project>
gh repo create --source=. --remote=origin --push --private  # or --public
```

Toggle visibility later:

```bash
gh repo edit --visibility public  # or private
```

</details>

> [!TIP]
> **Review before you push.** In Claude Code, run `/code-review` to scan your changes for bugs and cleanups before committing. `/code-review ultra` launches a deep multi-agent cloud review of the current branch (or a PR number) — user-triggered and billed.

## What to back up

**Commit:**

- Source code
- `CLAUDE.md` / `AGENTS.md` and project `.claude/settings.json`
- Notes and drafts
- Small reference data (under ~10 MB)
- Figures, tables, and outputs you reference in writing

**Don't commit:**

- `.claude/settings.local.json` (already gitignored)
- Large data — use Git LFS, S3, or external storage
- Huge intermediate artifacts (regenerate from code as needed)
- Secrets: `.env`, API keys, credentials — keep these in `.gitignore`

Your `.gitignore` is your friend. Start with [github/gitignore](https://github.com/github/gitignore) templates for your language.

## Installing more tools

Beyond `git` and `gh`, the AI installs any tool the same way — paste a repo URL (anything from [awesome-ai-for-economists](https://github.com/hanlulong/awesome-ai-for-economists) works) and ask *"install this."* See [Step 4 — Work with research software](./software.md) for the full pattern.

[← Back to README](../README.md) — you're at the end of the guide. Time to actually do some research.
