# Step 3 — Typical research workflow (detail)

A repeatable daily flow that uses the AI without breaking your stride.

[← Back to README](../README.md)

## Contents

- [Open terminal at folder](#open-terminal-at-folder)
- [4-pane window layout](#4-pane-window-layout)
- [Daily flow](#daily-flow)
- [Subagents (within one session)](#subagents-within-one-session)

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

Run a separate AI agent in each pane — each working on a different project, file, or task in parallel. While one agent compiles, another iterates on a draft; a third cleans data while a fourth drafts a referee response.

```
┌──────────────────┬──────────────────┐
│  AI agent 1      │  AI agent 2      │
│  Project A       │  Project B       │
├──────────────────┼──────────────────┤
│  AI agent 3      │  AI agent 4      │
│  Project C       │  Task X          │
└──────────────────┴──────────────────┘
```

Each pane is an independent session — its own conversation history, its own working directory, its own model. Cost scales with usage, and subscription plans enforce rolling caps (see [managing usage](./configuration.md#reasoning-ultracode--fast-mode)).

### Easy way: OS window snapping

Open four separate terminal windows (each running `claude` or `codex` in a different project folder), then snap each to a quarter of the screen using your OS's built-in shortcuts. Simpler than learning terminal-specific pane keybindings, and each window is a fully independent process.

**Windows 11:**

| Shortcut                    | Action               |
|-----------------------------|----------------------|
| `Win+Left`                  | Snap to left half    |
| `Win+Right`                 | Snap to right half   |
| `Win+Left` then `Win+Up`    | Top-left quarter     |
| `Win+Left` then `Win+Down`  | Bottom-left quarter  |
| `Win+Right` then `Win+Up`   | Top-right quarter    |
| `Win+Right` then `Win+Down` | Bottom-right quarter |

**Mac:** macOS Sequoia (15+) has built-in half-screen tiling (`Fn+Ctrl+Left/Right/Up/Down`). For quarter-tiling, install [Rectangle](https://rectangleapp.com) — free, open-source, with sensible defaults out of the box.

### Windows Terminal panes (alternative)

Default bindings per [Microsoft Docs](https://learn.microsoft.com/en-us/windows/terminal/customize-settings/actions):

| Shortcut             | Action                                                  |
|----------------------|---------------------------------------------------------|
| `Alt+Shift+D`        | Duplicate current pane (auto-chooses split direction)   |
| `Alt+Shift+=`        | Split right (vertical divider, panes side-by-side)      |
| `Alt+Shift+-`        | Split down (horizontal divider, panes stacked)          |
| `Alt+Arrow`          | Move focus to adjacent pane                             |
| `Alt+Shift+Arrow`    | Resize current pane                                     |
| `Ctrl+Shift+T`       | New tab                                                 |
| `Ctrl+Shift+W`       | Close current pane                                      |

### iTerm2 panes (alternative)

| Shortcut          | Action                                                  |
|-------------------|---------------------------------------------------------|
| `Cmd+D`           | Split vertically (vertical divider, panes side-by-side) |
| `Cmd+Shift+D`     | Split horizontally (horizontal divider, panes stacked)  |
| `Cmd+]` / `Cmd+[` | Next / previous pane                                    |
| `Cmd+Opt+Arrow`   | Move focus to adjacent pane                             |
| `Cmd+T`           | New tab                                                 |
| `Cmd+W`           | Close current pane                                      |

> [!NOTE]
> Both terminals name splits after the *divider*, not the panes. "Split vertically" gives you side-by-side panes (the divider between them is vertical).

### Why four AI agents

- **Parallel projects.** One agent per project, each with its own context and folder. Switch between them at human speed; never lose state.
- **Plan + execute.** One in plan mode (read-only) for design and research, the other in `auto` mode for execution — it runs lower-risk tool calls automatically and blocks the rest. They coordinate via shared files (one writes `plan.md`, the other implements).
- **Model comparison.** Send the same hard problem to two different models (one Codex, one Claude) — pick whichever answers better. The two leapfrog each other, so re-run the comparison periodically.

## Daily flow

1. **Open project folder + 4-pane layout** (see above).
2. **Resume yesterday's session:**
   - `claude -c` — resume most recent in this dir
   - `codex resume --last` — resume most recent in cwd (or `codex resume` for an interactive picker)
3. **State today's goal** in one sentence. The AI does better with a North Star.
4. **Let it work.** Use Tab (Codex) to add context without interrupting the running turn. See [configuration.md](./configuration.md#keyboard-shortcuts) for the Tab vs Enter rule.
5. **Manage context as it fills.** `/compact` condenses history while keeping direction on the *same* task. When you *switch* topics, `/clear` (Claude Code) or a fresh `codex` session starts clean — stale context otherwise leaks in and burns tokens.
6. **Commit at end of day.** Small commits, descriptive messages. Ask the AI to draft the message — it's usually good at this.

> [!TIP]
> **Ask the AI to double-check after it finishes.** When the AI completes a complex task — derivation, analysis, paper section, code change — send a brief follow-up: *"Double-check everything is correct. Zero errors. Improve. Make everything professional."* The AI re-examines its own output and catches mistakes, polishes weak spots, and raises the quality bar. One of the cheapest accuracy boosts available. To dial effort up or down: in Claude Code, raise reasoning with `/effort max` or `/effort ultracode` (xhigh + automatic multi-agent workflows; see [configuration.md](./configuration.md#reasoning-ultracode--fast-mode)), and drop to `/fast` or a lower level for routine edits; in Codex, raise or lower `model_reasoning_effort` (or `codex --profile fast` for quick work).

## Subagents (within one session)

Distinct from the 4-pane setup above (where you run several AI sessions in parallel), each session can also internally spawn **subagents** to handle independent parts of a single task. Useful for:

- Researching multiple datasets at once
- Reviewing several files in parallel
- Running independent experiments

In Claude Code, the orchestrator delegates via the `Agent` tool — specialized subagent types like `Explore` handle parallel code search and review. For larger, structured fan-outs it can also run deterministic multi-agent **workflows** via the Workflow tool (Dynamic Workflows — default on for Max/Team/Enterprise, off for Pro; toggle in `/config`), and [ultracode](./configuration.md#reasoning-ultracode--fast-mode) turns this on automatically for substantive tasks. Codex CLI has comparable parallel-agent features; check the current `codex` docs for the exact syntax.

> [!TIP]
> **Don't over-delegate.** Subagents help when work is genuinely parallel and independent. For sequential work, one focused agent beats three confused ones. The cost (in context and in your time managing them) only pays off when you can run 2+ tasks at the same time without dependencies between them.

→ Next: [Step 4 — Research software](./software.md)
