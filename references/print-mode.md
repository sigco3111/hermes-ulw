# Print Mode — Deep Dive

## Why Print Mode

Print mode (`claude -p`) is the **lowest-overhead** way to invoke Claude Code:

- **No PTY needed** → no terminal dialog handling
- **No tmux required** → no persistent session management
- **Exit when done** → no manual `/exit`
- **JSON output available** → structured results for piping
- **Cost bounded** → `--max-budget-usd` cap

Use print mode whenever the task is **self-contained** — you don't need to follow up on what Claude does mid-work.

## The Core Pattern

```bash
zsh -i -c 'claude -p "<TASK>" --max-turns <N> [other flags]'
```

The `zsh -i -c` wrapper is **mandatory on macOS** (Claude Code needs `ANTHROPIC_API_KEY` from `~/.zshrc`). On Linux, you can usually skip it.

## All Flags Reference

### Task Limits

| Flag | Default | When to set |
|---|---|---|
| `--max-turns N` | unlimited | Always — start with 5-10 |
| `--max-budget-usd X` | unlimited | Production use — set $0.50-2.00 |
| `--effort low\|medium\|high\|max\|auto` | auto | Override Claude's reasoning depth |

### Model Selection

| Flag | When |
|---|---|
| `--model haiku` | Trivial tasks (cheapest, ~10× cheaper than sonnet) |
| `--model sonnet` | Default — most tasks |
| `--model opus` | Deep reasoning, complex multi-step |
| `--fallback-model <name>` | Auto-fallback when overloaded |

### Tools & Permissions

| Flag | Format | Effect |
|---|---|---|
| `--allowedTools "X,Y"` | Comma list | Whitelist tools |
| `--allowedTools "Read"` | Single | Read-only mode |
| `--allowedTools "Bash(git:*)"` | Bash scoped | Only allow git commands |
| `--disallowedTools "X,Y"` | Comma list | Blacklist specific tools |
| `--tools ""` | Empty | Disable all built-in tools |
| `--permission-mode <mode>` | default, acceptEdits, plan, bypassPermissions | Permission style |

> 🔒 **Security tip**: Use `--allowedTools "Read"` for review-only tasks; use `Bash(git:*)` patterns to limit blast radius.

### Output Format

| Flag | Format | Use |
|---|---|---|
| `--output-format text` | Plain text | Default |
| `--output-format json` | Single JSON result | Programmatic parsing |
| `--output-format stream-json` | Newline-delimited JSON | Real-time streaming |
| `--json-schema '...'` | JSON Schema string | Force structured output |
| `--verbose` | More logging | Debugging |
| `--include-partial-messages` | Streaming | Real-time partial output |

### Input Format

| Flag | Format | Use |
|---|---|---|
| `--input-format stream-json` | Streamed JSON | Bidirectional streaming |

### Context & Session

| Flag | Effect |
|---|---|
| `--session-id <uuid>` | Use specific session ID |
| `-c, --continue` | Resume most recent session in current dir |
| `-r, --resume <id>` | Resume specific session by ID |
| `--fork-session` | When resuming, fork to new ID |
| `--no-session-persistence` | Don't save session (CI/scripts) |

### Prompt & System

| Flag | Effect |
|---|---|
| `--append-system-prompt <text>` | Add to default system prompt |
| `--append-system-prompt-file <path>` | Add file contents to prompt |
| `--bare` | Skip plugins, hooks, MCP discovery (fastest CI) |
| `--agents '<json>'` | Define custom subagents inline |
| `--mcp-config <path>` | Load MCP servers |

### Worktree Isolation

| Flag | Effect |
|---|---|
| `-w, --worktree [name]` | Run in `.claude/worktrees/<name>` |
| `--tmux` | Create tmux for worktree (with `-w`) |

### Debugging

| Flag | Effect |
|---|---|
| `-d, --debug` | Enable debug logging |
| `--debug-file <path>` | Write debug logs |

## Piping Input

### File via stdin

```bash
cat src/auth.py | claude -p "Review this code for bugs" --max-turns 1
```

### Multiple files

```bash
cat src/*.py | claude -p "List all TODO comments" --max-turns 1
```

### Git diff review

```bash
git diff HEAD~3 | claude -p "Summarize changes" --max-turns 1
```

### Command output

```bash
npm test 2>&1 | claude -p "Analyze failures" --max-turns 1
```

## Structured Output (JSON)

### Force a schema

```bash
claude -p 'List all functions in src/' \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}' \
  --max-turns 5
```

Returns:

```json
{
  "result": "{\"functions\": [\"foo\", \"bar\"]}",
  "structured_output": {"functions": ["foo", "bar"]},
  "session_id": "...",
  "num_turns": 3,
  "total_cost_usd": 0.045,
  "duration_ms": 12345,
  "stop_reason": "end_turn"
}
```

### Streaming JSON (real-time)

```bash
claude -p "Write a summary" \
  --output-format stream-json --verbose --include-partial-messages
```

Filter with jq:

```bash
claude -p "..." --output-format stream-json --verbose --include-partial-messages \
  | jq -rj 'select(.type == "stream_event" and .event.delta.type? == "text_delta") | .event.delta.text'
```

## Cost Optimization Patterns

### Pattern 1: haiku for trivial work

```bash
zsh -i -c 'claude -p "이 함수의 docstring 작성해줘" --model haiku --max-turns 3'
```

~10× cheaper than sonnet, sufficient for boilerplate.

### Pattern 2: sonnet for everything else

```bash
zsh -i -c 'claude -p "버그 찾아서 고쳐줘" --model sonnet --max-turns 10'
```

Good balance for ~90% of tasks.

### Pattern 3: opus only for deep reasoning

```bash
zsh -i -c 'claude -p "이 알고리즘의 시간복잡도 최적화해줘" --model opus --max-turns 15'
```

5× sonnet price; reserve for hard problems.

## Bare Mode (CI/Scripting)

```bash
claude --bare -p "Run all tests" --allowedTools 'Read,Bash' --max-turns 10
```

Skips hooks, plugins, MCP discovery, CLAUDE.md loading.
Requires `ANTHROPIC_API_KEY` env (OAuth skipped).

## Session Resumption

```bash
# Start task, capture session ID
claude -p "Start refactor" --output-format json --max-turns 10 > /tmp/s.json

# Resume with captured ID
SID=$(python3 -c "import json; print(json.load(open('/tmp/s.json'))['session_id'])")
claude -p "Continue" --resume "$SID" --max-turns 5
```

Useful for: multi-step migrations, multi-day PRs.

## Common Recipes

### PR review from diff

```bash
gh pr diff <num> | claude -p "Review thoroughly. Security, bugs, style." --max-turns 3
```

### Auto-generate commit message

```bash
git diff --staged | claude -p "Generate conventional commit message" --max-turns 1
```

### Find TODOs

```bash
grep -r "TODO" src/ | claude -p "Group TODOs by file/module" --max-turns 2
```

### Generate tests

```bash
cat src/auth.py | claude -p "Generate pytest tests for public functions" --max-turns 8 \
  --allowedTools "Read,Write,Bash"
```

## When Print Mode Fails

| Symptom | Cause | Fix |
|---|---|---|
| "Invalid API key" | zshrc not sourced | Add `zsh -i -c '...'` wrapper |
| "Tool use not allowed" | Tool not in whitelist | Add to `--allowedTools` |
| Hits max-turns early | Task too complex | Increase `--max-turns` or split task |
| Output truncated | max-tokens on long output | Use `--output-format json` |
| Cost too high | Opus used for trivial work | Use `--model haiku` |
