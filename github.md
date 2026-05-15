# GitHub setup and configuration

Both AI tools (Codex CLI and Claude Code) assume you have **git** and the **GitHub CLI** (`gh`) installed and authenticated. `gh auth login` is the easiest path — it handles HTTPS tokens *and* SSH keys in one interactive flow, and gives Claude Code / Codex CLI a working `gh` for PR and issue commands out of the box.

[← Back to README](./README.md)

## Install

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

Set your identity and a few sensible global defaults, then authenticate. Run these in any terminal:

```bash
# Identity — used as your commit author
git config --global user.name  "Your Name"
git config --global user.email you@example.com

# Sensible defaults
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global push.autoSetupRemote true

# Authenticate gh — walks you through HTTPS token + SSH key upload
gh auth login
```

### The `gh auth login` flow

It asks four things:

1. **GitHub.com or GitHub Enterprise?** → GitHub.com
2. **Preferred protocol for git operations?** → SSH (recommended; `gh` will offer to generate a key and upload it for you)
3. **Upload your SSH public key to your GitHub account?** → Yes
4. **How would you like to authenticate?** → Login with a web browser

A one-time code is shown; press Enter and your browser opens. Approve, return to the terminal, and you're done.

### Verify

```bash
gh auth status                # should show "Logged in to github.com as <you>"
gh repo view <your-username>  # any public repo of yours
ssh -T git@github.com         # should say "Hi <you>! You've successfully authenticated..."
```

After this, `git clone git@github.com:...`, `gh repo create`, `gh pr create`, and AI-tool GitHub integrations all work without further setup.

## What's next

- **Per-repo identity** if you maintain a separate work/personal split: in the repo, `git config user.email work@example.com` overrides the global setting.
- **Commit signing** (SSH-key signing is now first-class on GitHub and reuses the key you just uploaded) — to be covered in a later sub-page.
- **Useful aliases** (`git lg`, `git co`, etc.) — also for a later sub-page.
