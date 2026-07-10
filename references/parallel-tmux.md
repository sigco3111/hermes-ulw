# Parallel tmux — Deep Dive

## Why Parallel tmux

Sometimes the fastest path is **2-3 N concurrent tasks**. Instead of:

```bash
# Serial: 45 minutes total
claude -p "fix bug"     # ~15 min
claude -p "add tests"   # ~15 min
claude -p "update docs" # ~15 min
```

You run:

```bash
# Parallel: ~15 minutes total
tmux send-keys -t task1 ...
tmux send-keys -t task2 ...
tmux send-keys -t task3 ...
```

**Trade-offs:**
- ✅ Wall-clock speedup (3× for 3 independent tasks)
- ✅ True task isolation (each Claude has its own context)
- ⚠️ Cost is multiplied (3× API spend)
- ⚠️ Coordination required (which task finished?)

## When to Use Parallel Mode

**Use parallel when:**
- ✅ 2-5 tasks are TRULY independent (different files, no shared state)
- ✅ Each task takes > 5 minutes (parallel overhead worth it)
- ✅ Cost is acceptable (multi-claude invocation)
- ✅ Tasks can be verified independently

**Don't parallelize:**
- ❌ Tasks that edit the same files (race conditions)
- ❌ Tasks that depend on each other (sequential)
- ❌ Tasks that are < 2 minutes (overhead)
- ❌ When one task's result informs the others

## The Basic Pattern

```bash
# 1. Create N sessions (one per task)
for i in 1 2 3; do
  tmux new-session -d -s "task$i" -x 140 -y 40
done

# 2. Launch each task with its own Claude Code instance
tmux send-keys -t task1 'cd ~/Developer/MyApp && zsh -i -c "claude -p \"auth 버그 수정\" --allowedTools \"Read,Edit\" --max-turns 10"' Enter

tmux send-keys -t task2 'cd ~/Developer/MyApp && zsh -i -c "claude -p \"API 테스트 추가\" --allowedTools \"Read,Write,Bash\" --max-turns 15"' Enter

tmux send-keys -t task3 'cd ~/Developer/MyApp && zsh -i -c "claude -p \"README 업데이트\" --allowedTools \"Read,Edit\" --max-turns 5"' Enter

# 3. Monitor all in parallel
sleep 30 && for s in task1 task2 task3; do
  echo "=== $s ==="
  tmux capture-pane -t $s -p -S -10
done

# 4. Cleanup
for s in task1 task2 task3; do
  tmux send-keys -t $s '/exit' Enter
done
sleep 5
for s in task1 task2 task3; do
  tmux kill-session -t $s 2>/dev/null
done
```

## Worktree-Based Parallel (Safer)

**The risk:** Two Claudes editing the same file = conflict.

**The fix:** Each Claude gets its own git worktree.

```bash
# task1 edits feature/auth in its worktree
tmux send-keys -t task1 'cd ~/Developer/MyApp && zsh -i -c "claude -p \"auth 버그 수정\" -w auth-bug"' Enter

# task2 edits feature/tests in its worktree
tmux send-keys -t task2 'cd ~/Developer/MyApp && zsh -i -c "claude -p \"API 테스트 추가\" -w test-add"' Enter

# task3 edits feature/docs in its worktree
tmux send-keys -t task3 'cd ~/Developer/MyApp && zsh -i -c "claude -p \"README 업데이트\" -w docs"' Enter
```

Each Claude sees only its worktree's files. No conflicts.

After all complete, you merge worktrees:
```bash
cd ~/Developer/MyApp
git merge feature/auth-bug
git merge feature/test-add
git merge feature/docs
```

## Race Condition Avoidance

### Safe Patterns

1. **Different file scopes per task**
   - task1: `src/auth/`
   - task2: `src/api/`
   - task3: `docs/`

2. **Different file types per task**
   - task1: `*.ts` files
   - task2: `*.py` files
   - task3: `*.md` files

3. **Read-only + write split**
   - task1 writes (Read, Edit)
   - task2 reads only (Read)

### Unsafe Patterns (Avoid)

1. **Same file edited by two tasks** → last writer wins, surprise
2. **Task A depends on Task B's output** → use serial
3. **Tasks that touch package.json/lockfile** → only one should manage deps

## Fan-out / Fan-in

Sometimes you want Claude to dispatch tasks to other Claudes (delegation).

### Inside one Claude session
```bash
# Inside Claude REPL:
"Use the @reviewer agent to check this PR, then @tester agent to write tests"
```

### Across tmux sessions
```bash
# Master session watches slaves
tmux new-session -d -s master
tmux new-session -d -s slave1
tmux new-session -d -s slave2

# Master sends tasks
tmux send-keys -t slave1 'zsh -i -c "cd ~/p && claude -p \"task1\""' Enter
tmux send-keys -t slave2 'zsh -i -c "cd ~/p && claude -p \"task2\""' Enter

# Master polls slaves
tmux send-keys -t master 'zsh -i -c "while true; do
  echo \"=== slave1 ===\"
  tmux capture-pane -t slave1 -p -S -5
  echo \"=== slave2 ===\"
  tmux capture-pane -t slave2 -p -S -5
  sleep 30
done"' Enter
```

## Cost Calculation

| Tasks | Avg cost each | Total |
|---|---|---|
| 3 (sonnet, ~10 turns each) | $0.30 | **$0.90** |
| 3 (opus, ~15 turns each) | $1.50 | **$4.50** |
| 5 (haiku, ~3 turns each) | $0.02 | **$0.10** |

> For small tasks, parallel haiku is often the best cost-efficiency choice.

## Monitoring All Sessions

```bash
# Create helper function
ulw-poll() {
  for s in "$@"; do
    echo "=== $s ($(date +%T)) ==="
    tmux capture-pane -t "$s" -p -S -10 2>/dev/null || echo "  (session gone)"
  done
}

# Use it
ulw-poll task1 task2 task3
```

**Loop mode:**
```bash
for i in $(seq 1 20); do
  clear
  ulw-poll task1 task2 task3
  sleep 15
done
```

## Cleanup Discipline

Parallel tmux **leaks sessions** if not cleaned. Always end with:

```bash
for s in task1 task2 task3; do
  tmux send-keys -t "$s" '/exit' Enter
  sleep 1
  tmux kill-session -t "$s" 2>/dev/null
done

# Audit any zombies
tmux ls
```

## Common Pitfalls

1. **Two Claudes editing the same file** → use worktree per task
2. **Zombie tmux sessions** → always clean up
3. **Cost surprise** → opus × 5 sessions × long turns = $$$; budget early
4. **Output explosion in capture-pane** → limit with `-S` flag
5. **Sequential wait when parallel was wanted** → poll with `sleep` correctly
6. **Trust dialog spam** → first session handles trust for that dir; subsequent don't show

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Two Claudes modifying same file | No worktree isolation | Use `claude -w <name>` per task |
| One task stalls forever | Unclear task | Cancel + smaller scope |
| Session produces garbage output | `--max-turns` too low | Raise to 15-20 |
| Cost too high | Wrong model | Use `haiku` for trivial tasks |
| Output duplicated across tasks | Tasks not actually independent | Restructure or serialize |
