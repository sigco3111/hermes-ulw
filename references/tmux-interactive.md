# tmux Interactive Mode — Deep Dive

## Why tmux Interactive

Print mode is great for single-shot tasks, but some workflows **need**:

- Multi-turn iteration (you review → follow up → continue)
- Slash commands (`/compact`, `/review`)
- Human-in-the-loop decisions
- Watching Claude work in real time
- A persistent context that survives across your follow-ups

**tmux Interactive** = Claude Code runs in a persistent TUI inside a tmux session. You monitor with `capture-pane`, send new instructions with `send-keys`.

> Think of tmux as a **virtual terminal window you can script**. Claude Code is the app inside that window.

## Setting Up

### 1. Create a tmux session

```bash
tmux new-session -d -s claude-work -x 140 -y 40
```

- `-d` detach (run in background)
- `-s <name>` session name (use meaningful names like `claude-feature-xyz`)
- `-x 140 -y 40` terminal size (wide enough for Claude's TUI)

### 2. Launch Claude Code inside

```bash
tmux send-keys -t claude-work 'cd ~/Developer/MyApp && zsh -i -c "claude"' Enter
```

The `zsh -i -c` wrapping makes Claude see `ANTHROPIC_API_KEY` from `~/.zshrc`.

### 3. Trust dialog handling

On the **first** launch in a directory, Claude asks:
```
❯ 1. Yes, I trust this folder
  2. No, exit
```
Just press `Enter` — option 1 is the default.

```bash
sleep 5 && tmux send-keys -t claude-work Enter
```

> Trust is cached for the directory; this dialog only appears once per dir.

### 4. Send your initial task

```bash
tmux send-keys -t claude-work 'JWT 토큰 쓰도록 auth 모듈 리팩토링해줘' Enter
```

### 5. Monitor progress

```bash
sleep 15 && tmux capture-pane -t claude-work -p -S -50
```

`-S -50` means "show last 50 lines". Adjust as needed.

### 6. Send follow-ups

```bash
tmux send-keys -t claude-work '이제 새 JWT 코드 유닛테스트 추가해줘' Enter
```

### 7. End the session

```bash
tmux send-keys -t claude-work '/exit' Enter
tmux kill-session -t claude-work
```

## Slash Commands

Inside the interactive REPL, you can use slash commands:

| Command | Effect |
|---|---|
| `/help` | Show all commands |
| `/compact [focus]` | Compress context (saves tokens; CLAUDE.md survives) |
| `/clear` | Wipe conversation history |
| `/context` | Visualize context usage (colored grid) |
| `/cost` | Per-model cost breakdown |
| `/resume` | Switch to different session |
| `/model [model]` | Switch model mid-session |
| `/effort [level]` | Adjust reasoning effort |
| `/init` | Create CLAUDE.md for project memory |
| `/memory` | Edit CLAUDE.md |
| `/config` | Settings picker |
| `/permissions` | View/update tool permissions |
| `/agents` | Manage subagents |
| `/mcp` | Manage MCP servers UI |
| `/review` | Request code review |
| `/security-review` | Security audit |
| `/plan` | Enter Plan mode |
| `/batch` | Auto-create worktrees for parallel changes |
| `/exit` or `Ctrl+D` | End session |

### Context Health Thresholds

- **< 70%** — Normal, full precision
- **70-85%** — Precision starts dropping → run `/compact`
- **> 85%** — Hallucination risk → `/compact` or `/clear`

## Keyboard Shortcuts (Inside TUI)

### General

| Key | Action |
|---|---|
| `Ctrl+C` | Cancel current input/generation |
| `Ctrl+D` | Exit session |
| `Ctrl+R` | Reverse search history |
| `Ctrl+B` | Background current task |
| `Ctrl+V` | Paste image |
| `Ctrl+O` | Transcript mode (show thinking) |
| `Esc Esc` | Rewind / summarize |

### Mode Toggles

| Key | Action |
|---|---|
| `Shift+Tab` | Cycle permission modes |
| `Alt+P` | Switch model |
| `Alt+T` | Toggle thinking mode |
| `Alt+O` | Toggle Fast Mode |

### Input Prefixes

| Prefix | Meaning |
|---|---|
| `!` | Execute bash directly, bypassing AI |
| `@` | Reference files/dirs (autocomplete) |
| `#` | Quick-add to CLAUDE.md memory |
| `/` | Slash commands |

### Special Keyword

The keyword **"ultrathink"** in any prompt triggers maximum reasoning effort for that turn (regardless of `/effort` setting).

## Reading the TUI Status

When monitoring via `capture-pane`, look for:

| Indicator | Meaning |
|---|---|
| `❯` at bottom | Waiting for your input |
| `●` lines | Actively using tools (Read, Edit, Bash) |
| `⏵⏵ bypass permissions on` | Auto-approve mode is on |
| `◐ medium · /effort` | Current effort level |
| `ctrl+o to expand` | Output was truncated |

```bash
# Quick poll every 10s
for i in 1 2 3 4 5 6; do
  echo "=== $(date +%T) ==="
  tmux capture-pane -t claude-work -p -S -5
  sleep 10
done
```

## Worktree Isolation

For multi-PR parallel work, use git worktree:

```bash
tmux send-keys -t claude-feature 'cd ~/Developer/MyApp && claude -w feature-auth' Enter
```

This creates `.claude/worktrees/feature-auth/` for the work, isolated from main checkout.

### Auto-tmux worktree

```bash
claude -w feature-xyz --tmux
```

Spawns a tmux session for the worktree automatically.

## Multi-Turn Pattern: Plan → Implement → Review → Fix

```bash
# 1. Start session
tmux new-session -d -s dev -x 140 -y 40
tmux send-keys -t dev 'cd ~/Developer/MyApp && zsh -i -c "claude"' Enter
sleep 5 && tmux send-keys -t dev Enter  # Trust dialog

# 2. Plan mode
tmux send-keys -t dev '/plan auth 모듈 JWT 마이그레이션 단계별로 계획해줘' Enter
sleep 20 && tmux capture-pane -t dev -p -S -30

# 3. Implement
tmux send-keys -t dev '계획대로 구현 시작해줘' Enter

# 4. Periodic check + follow-up
sleep 30 && tmux capture-pane -t dev -p -S -10
tmux send-keys -t dev '테스트 추가해줘' Enter

# 5. Review
sleep 30 && tmux send-keys -t dev '/review' Enter

# 6. Final cleanup
tmux send-keys -t dev '/exit' Enter
tmux kill-session -t dev
```

## CLAUDE.md — Project Context File

Auto-loaded when Claude Code starts in a directory. Use it to persist project-specific knowledge.

### Project example

```markdown
# Project: My App

## Architecture
- FastAPI backend with SQLAlchemy ORM
- PostgreSQL, Redis cache
- pytest for testing, target 90% coverage

## Commands
- make test — full test suite
- make lint — ruff + mypy
- make dev — start dev server :8000

## Standards
- Type hints on all public functions
- 2-space YAML, 4-space Python
- No wildcard imports
```

Place at:
- Global: `~/.claude/CLAUDE.md`
- Project: `./CLAUDE.md` (git-tracked)
- Local: `.claude/CLAUDE.local.md` (gitignored)

For many rules, use `.claude/rules/*.md` (modular).

## Memory Persistence (Auto)

Claude Code auto-stores learned context in `~/.claude/projects/<project>/memory/`:
- **Limit:** 25KB or 200 lines per project
- Separate from CLAUDE.md

## Common Pitfalls

1. **Forgetting trust dialog handling** — interactive mode appears frozen if you don't press Enter.
2. **Trust dialog appears AGAIN** even with same dir if you moved/different sessions — re-handle.
3. **`/compact` loses critical context** — only run when context > 70%.
4. **`/clear` erases everything** — only for fresh starts.
5. **Multiple tmux sessions growing** — `tmux ls` to audit, then `kill-session` for cleanup.
6. **Background tmux from logout** — `-d` flags correctly handle this; check `tmux ls` periodically.
7. **Reading context-saturated session** — `/cost` first, then plan `/compact`.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Session appears frozen | Trust dialog not handled | `tmux send-keys -t <name> Enter` |
| `claude` command not found | PATH missing | `which claude`; check `~/.local/bin` |
| "Permission denied" for files | `--permission-mode` | Use `bypassPermissions` or `--allowedTools` |
| Auth fails mid-session | Token expired | `/login` inside TUI |
| Output scroll broken | Terminal too small | Recreate with `-x 140 -y 40` |
