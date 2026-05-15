# Step 3 — Typical research workflow (detail)

A repeatable daily flow that uses the AI without breaking your stride.

[← Back to README](../README.md)

## Contents

- [Open terminal at folder](#open-terminal-at-folder)
- [4-pane window layout](#4-pane-window-layout)
- [Daily flow](#daily-flow)
- [Parallel agents](#parallel-agents)

## Open terminal at folder

### Windows

In File Explorer, navigate to the project folder. Then type one of these in the **address bar** and press Enter:

- `cmd` — opens CMD in this folder
- `powershell` — opens PowerShell here
- `wt` — opens Windows Terminal (default profile, in this folder)

`wt` is the most modern choice if Windows Terminal is installed (it ships with Windows 11).

### Mac

Several paths, pick whichever is fastest for you:

- **Finder right-click:** right-click the folder → Services → New Terminal at Folder. If you don't see it, enable in **System Settings → Keyboard → Keyboard Shortcuts → Services → Files and Folders → New Terminal at Folder**.
- **iTerm2:** drag the Finder folder onto the iTerm2 dock icon — opens a tab in that folder.
- **From an open terminal:** `cd <path>` (tab-completes).

## 4-pane window layout

A typical layout for AI-assisted research:

```
┌──────────────────┬──────────────────┐
│  Editor          │  Browser /       │
│  (VS Code,       │  Notes /         │
│   Cursor)        │  Reference docs  │
├──────────────────┼──────────────────┤
│  Codex CLI       │  Claude Code     │
└──────────────────┴──────────────────┘
```

### iTerm2 (Mac) — pane shortcuts

| Shortcut         | Action                          |
|------------------|---------------------------------|
| `Cmd+D`          | Split pane vertically (right)   |
| `Cmd+Shift+D`    | Split pane horizontally (below) |
| `Cmd+[` / `Cmd+]`| Move focus between panes        |
| `Cmd+T`          | New tab                         |
| `Cmd+W`          | Close current pane              |

### Windows Terminal — pane shortcuts

| Shortcut                 | Action                              |
|--------------------------|-------------------------------------|
| `Alt+Shift+D`            | Duplicate pane (auto-split)         |
| `Alt+Shift+Plus`         | Split pane vertically               |
| `Alt+Shift+Minus`        | Split pane horizontally             |
| `Alt+` arrow             | Move focus between panes            |
| `Ctrl+Shift+T`           | New tab                             |
| `Ctrl+Shift+W`           | Close current pane                  |

### Why two AI tools side-by-side

- **Compare same task in parallel.** Ask both about the same hard problem; whichever gives the better answer wins. Fastest signal on which model handles your specific kind of work.
- **One for plan, one for execution.** Use Claude Code's plan mode for design and research; use Codex with `--permission-mode auto` for the actual edits.
- **Context isolation.** One tool keeps the long-running task; the other handles small fast lookups without polluting the main context.

## Daily flow

1. **Open project folder + 4-pane layout** (see above).
2. **Resume yesterday's session:**
   - `claude -c` — resume most recent in this dir
   - `codex --resume` — pick from prior sessions
3. **State today's goal** in one sentence. The AI does better with a North Star.
4. **Let it work.** Use Tab (Codex) to add context without interrupting the running turn. See [configuration.md](./configuration.md#keyboard-shortcuts) for the Tab vs Enter rule.
5. **Compact when context fills up.** In Claude Code, `/compact` condenses history while preserving direction.
6. **Commit at end of day.** Small commits, descriptive messages. Ask the AI to draft the message — it's usually good at this.

## Parallel agents

Both tools can spawn subagents to work on independent parts of a task in parallel. Useful for:

- Researching multiple datasets at once
- Reviewing several files in parallel
- Running independent experiments

In Claude Code, the orchestrator delegates via the `Agent` tool (and there are specialized subagent types like `Explore` for code search). In Codex, the equivalent is the `multi_agent` config and the `/agents` slash command.

> [!TIP]
> **Don't over-delegate.** Subagents help when work is genuinely parallel and independent. For sequential work, one focused agent beats three confused ones. The cost (in context and in your time managing them) only pays off when you can run 2+ tasks at the same time without dependencies between them.

→ Next: [Step 4 — Research software](./software.md)
