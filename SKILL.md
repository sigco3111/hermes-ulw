---
name: hermes-ulw
description: "Use when the user mentions 'ULW', 'OmniCoder ULW', 'Omni-Coder Ultra-Large-Window', or asks to delegate a coding task using their previously-familiar Ultra-Large Window workflow. Routes the task to Claude Code in Hermes using one of three modes (print mode for single-shot tasks, tmux interactive for multi-turn work, or parallel tmux for concurrent jobs) and auto-selects the mode based on the task's shape — single-shot vs iterative vs parallel-concurrent. Replaces the Omni-Coder ULW workflow with Claude Code's CLI."
version: 0.1.0
author: hjshin (sigco3111)
license: MIT
platforms: [linux, macos]
metadata:
  hermes:
    tags: [Coding-Agent, Claude-Code, OmniCoder-ULW, Workflow, Orchestration, PTY, tmux, Print-Mode, Parallel]
    related_skills: [claude-code, codex, opencode]
---

# Hermes ULW — Omni-Coder ULW Workflow Replacement

## Overview

Omni-Coder's ULW (Ultra-Large Window) gave you a single big context where one AI handled coding tasks top-to-bottom. **Hermes ULW** delivers the same delegating-to-AI experience using **Claude Code's CLI**, but with three distinct execution modes that auto-route based on task shape. The result is the same outcome — coding tasks run in isolated context, you get a clean summary back — but with **better cost control** and **built-in parallel execution** that ULW never had.

> **Food analogy**: ULW was one giant kitchen handed to one chef. Hermes ULW is **three kitchens** — a single-shot kiosk, a multi-course restaurant, and a buffet — you pick based on the order size. Or you let the orchestrator pick for you.

## When to Use

**Trigger this skill when the user says any of:**
- "ulw로 해줘", "ulw 워크플로우로", "ulw mode로"
- "OmniCoder ULW", "Omni-Coder ULW", "Ultra-Large Window"
- "큰 컨텍스트로 코딩 위임", "격리된 환경에서 코딩 작업"
- "Claude Code print mode", "tmux 인터랙티브", "병렬 작업"
- "코딩 작업 위임", "큰 일 시켜", "delegating coding task"

**Don't trigger for:**
- Questions about Claude Code itself (use `claude-code` skill)
- Non-coding tasks (use general delegation)
- Quick edits (do them inline — don't over-delegate)

## Mode Selection Decision Tree

The orchestrator (that's you) chooses a mode **before** invoking Claude Code. Three questions:

```
Is the task a SINGLE, well-defined action?
└── YES → Print Mode (one-shot, fast, $0.01-0.20)
    Examples: bug fix, code review, doc generation, refactor single file

Is the task ITERATIVE (multi-turn refinement, builds on previous output)?
└── YES → tmux Interactive (persistent REPL, mid-high cost)
    Examples: build a feature, debug interactively, plan-then-implement

Are there 2+ CONCURRENT INDEPENDENT tasks?
└── YES → Parallel tmux (3+ Claude Code instances)
    Examples: bug-fix + test-add + docs all at once
```

When in doubt, **default to Print Mode** — it's the lowest-cost and most predictable.

## Mode 1: Print Mode (Default)

The simplest mode. One terminal command, one result.

### Invocation

```bash
zsh -i -c 'claude -p "<TASK>" --max-turns <N> [--model haiku|sonnet|opus] [--allowedTools "<TOOLS>"] [other flags]'
```

> ⚠️ Wrap in `zsh -i -c '...'` for macOS to ensure `~/.zshrc` is sourced (Claude Code needs `ANTHROPIC_API_KEY` from zshrc).

### Common Flags

| Flag | Default | Purpose |
|---|---|---|
| `--max-turns N` | unlimited | Cap agentic loops (5-10 typical) |
| `--max-budget-usd X` | unlimited | Cap API spend ($0.50 typical) |
| `--model <name>` | sonnet | haiku (cheap), sonnet (default), opus (heavy) |
| `--allowedTools "<X,Y>"` | all | Whitelist allowed tools |
| `--output-format json` | text | Get structured result |
| `--json-schema '...'` | none | Force JSON schema output |

### Worked Example

```bash
# Fix the login bug
zsh -i -c 'cd ~/Developer/MyApp && claude -p "로그인 시 비밀번호 틀리면 에러 안 나는 버그를 찾아 고쳐줘" \
  --allowedTools "Read,Edit,Bash" --max-turns 10'
```

### When Print Mode Completes

You get back:
- A short text result (or JSON if requested)
- `session_id` — if you want to continue later
- `num_turns`, `total_cost_usd`, `duration_ms` — for cost tracking

## Mode 2: tmux Interactive

For tasks that need back-and-forth. Claude Code runs as a persistent TUI inside tmux; you monitor and add follow-up prompts.

### Invocation

```bash
# 1. Create tmux session
tmux new-session -d -s <NAME> -x 140 -y 40

# 2. Launch Claude Code inside (zsh -i for zshrc source)
tmux send-keys -t <NAME> 'cd ~/Developer/MyApp && zsh -i -c "claude"' Enter

# 3. Handle first-time trust dialog (default = Yes, just press Enter)
sleep 5 && tmux send-keys -t <NAME> Enter

# 4. Send task
tmux send-keys -t <NAME> 'JWT 토큰 쓰도록 auth 모듈 리팩토링해줘' Enter

# 5. Monitor
sleep 15 && tmux capture-pane -t <NAME> -p -S -50

# 6. Send follow-up
tmux send-keys -t <NAME> '이제 새 JWT 코드 유닛테스트 추가해줘' Enter

# 7. Cleanup
tmux send-keys -t <NAME> '/exit' Enter && tmux kill-session -t <NAME>
```

### Critical PTY Dialog Handling

Claude Code presents a **trust dialog** on first visit to a directory:
- `1. Yes, I trust this folder ← DEFAULT` → press Enter to accept

If using `--dangerously-skip-permissions`, a **second dialog** appears:
- `1. No, exit ← DEFAULT (WRONG)` → must press Down then Enter

> 💡 **Tip**: Just use print mode with `--allowedTools` to skip these dialogs entirely.

### When to Use Interactive Mode

- Multi-turn refactor → review → fix → test cycle
- Tasks requiring human-in-the-loop decisions
- When using slash commands (`/compact`, `/review`)
- Exploration (large unfamiliar codebase)

### Slash Commands Cheat Sheet

| Command | Use |
|---|---|
| `/compact` | Compress context to save tokens |
| `/clear` | Wipe history (fresh start) |
| `/context` | Visualize context usage |
| `/cost` | Token usage breakdown |
| `/resume` | Switch sessions |
| `/exit` | End session |
| `# <note>` | Add to CLAUDE.md memory |

## Mode 3: Parallel tmux

For 2+ independent tasks running simultaneously.

### Invocation

```bash
# Create N sessions
for i in 1 2 3; do
  tmux new-session -d -s "task$i" -x 140 -y 40
done

# Spawn each with independent task
tmux send-keys -t task1 'cd ~/Developer/MyApp && zsh -i -c "claude -p \"auth 버그 수정\" --allowedTools \"Read,Edit\" --max-turns 10"' Enter

tmux send-keys -t task2 'cd ~/Developer/MyApp && zsh -i -c "claude -p \"API 테스트 추가\" --allowedTools \"Read,Write,Bash\" --max-turns 15"' Enter

tmux send-keys -t task3 'cd ~/Developer/MyApp && zsh -i -c "claude -p \"README 업데이트\" --allowedTools \"Read,Edit\" --max-turns 5"' Enter

# Monitor all
sleep 30 && for s in task1 task2 task3; do
  echo "=== $s ===" && tmux capture-pane -t $s -p -S -10
done
```

### When to Use Parallel Mode

- 2+ tasks truly independent (no shared state, no communication)
- Wall-clock time matters (3 jobs in 15 min vs 45 min serial)
- Cost is acceptable (multi-claude invocation = multi-billing)

**Don't parallelize tasks that:**
- Edit the same files (conflict risk)
- Need results of each other (sequential dependency)
- Are cheap enough that print mode does fine in series

## Common Pitfalls

1. **Forgetting `zsh -i -c`** on macOS — Claude Code won't find `ANTHROPIC_API_KEY` and crashes with "Invalid API key". Always wrap.
2. **Setting `--max-turns` too low** — silently truncates work. Start at 5-10 for print, no cap for interactive.
3. **Not handling the trust dialog** on first tmux launch — interactive mode appears frozen; just press Enter.
4. **Parallel tmux bombing the same files** — race condition. Use git worktree per task (`claude -w <name>`) for isolation.
5. **Cost runaway on opus model** — opus is 5× sonnet price. Default to sonnet unless task needs deep reasoning.
6. **`ANTHROPIC_BASE_URL` not set when using proxy** (e.g., minimax, OpenRouter) — see references/zshrc-setup.md.
7. **Token key in plain `.zshrc` with no `chmod`** — if macOS backup syncs the file, key leaks. Use 1Password CLI or macOS Keychain.

## Verification Checklist

After running any mode:

- [ ] Result came back without "Invalid API key" or "Tool use not allowed"
- [ ] `--max-turns` was hit before any auto-truncation (check `num_turns`)
- [ ] Cost is within budget (check `total_cost_usd` or set hard cap)
- [ ] For tmux: `tmux ls` shows sessions, no orphans
- [ ] For interactive: `/context` shows < 70% memory usage (else `/compact`)
- [ ] For parallel: all sessions completed (no zombie tmux)

## Quick Reference: One-Line Recipes

**Code review:**
```bash
zsh -i -c 'cd ~/Developer/MyApp && git diff main | claude -p "이 변경사항 리뷰" --max-turns 3'
```

**Bug fix:**
```bash
zsh -i -c 'cd ~/Developer/MyApp && claude -p "로그인 버튼 안 눌리는 버그 고쳐줘" --allowedTools "Read,Edit" --max-turns 10'
```

**Generate docs:**
```bash
zsh -i -c 'cd ~/Developer/MyApp && claude -p "src/ 구조 분석해서 README 만들어줘" --allowedTools "Read,Write" --max-turns 8'
```

**Refactor:**
```bash
zsh -i -c 'cd ~/Developer/MyApp && claude -p "auth 모듈을 react-hook-form으로 리팩토링" --allowedTools "Read,Edit" --max-turns 15'
```

## References

- `references/print-mode.md` — Deep dive on print mode flags, JSON schemas, piping input
- `references/tmux-interactive.md` — Multi-turn workflows, worktree isolation, slash commands
- `references/parallel-tmux.md` — Race condition avoidance, fan-out/fan-in patterns
- `references/zshrc-setup.md` — API key setup, base URLs, minimax/OpenRouter integration
- `templates/ulw-prompt-template.md` — Copy-paste templates for common task types

## One-Line Pitch

> **Hermes ULW = Omni-Coder ULW's job, done cheaper and parallel-ready, with Claude Code's CLI instead of ULW's giant context.**
