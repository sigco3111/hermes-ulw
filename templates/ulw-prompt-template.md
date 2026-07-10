# ULW Prompt Templates

Copy-paste these into the orchestrator (Hermes) to delegate work cleanly.

---

## Template 1: Single-Shot Print Mode

```
Run the following one-shot coding task via Print Mode:

Task: [한 줄로 명확하게]
Project path: [절대 경로]
Files: [수정해야 할 파일 범위]
Constraints:
  - Match existing code style (2-space indent, no wildcards)
  - Don't change unrelated files
  - Add minimal comments
Cost cap: $0.50
Max turns: 10
Model: sonnet (default)
Tools: Read, Edit (no Bash if not needed)

Run:
zsh -i -c 'cd "<PROJECT_PATH>" && claude -p "<TASK>" --allowedTools "Read,Edit" --max-turns 10 --max-budget-usd 0.5'

After completion, report:
- Did it succeed? (Y/N)
- Files changed
- Cost in USD
- num_turns used
```

---

## Template 2: Multi-Turn Interactive

```
Run this multi-turn coding task via tmux Interactive:

Task: [큰 기능 / 리팩토링]
Project path: [절대 경로]

Steps:
1. Create tmux session named "dev-<feature>"
2. Launch Claude Code there
3. Handle trust dialog (Enter)
4. Send initial task
5. Wait until planner + initial code is done, then send follow-ups

Initial task: [ex: "먼저 plan mode로 단계 정리해줘"]
Follow-up 1: [ex: "계획대로 구현 시작"]
Follow-up 2: [ex: "유닛테스트 추가"]
Follow-up 3: [ex: "/review 호출해서 리뷰"]

Cleanup: After all done, send /exit and kill-session.
Report back files changed, cost, duration.
```

---

## Template 3: Parallel Tasks

```
Run these N tasks in parallel via tmux:

Each task is independent. Each runs in its own tmux session.

Tasks:
1. [task1 description, files, constraints]
2. [task2 description, files, constraints]
3. [task3 description, files, constraints]

Use tmux sessions: task1, task2, task3
Each gets `--max-budget-usd 1.0` cap.
Each uses Claude Sonnet.
Use `claude -w <feature>` (worktree) per task to avoid file conflicts.

After all complete, capture summary from each:
- Files changed per task
- Cost per task
- num_turns per task
- Wall-clock total
```

---

## Template 4: Code Review

```
Run code review via Print Mode:

Project path: [절대 경로]
Base branch: main
Current branch: feature/xxx

Get the diff:
git diff main...HEAD

Pipe to Claude Code:
git diff main...HEAD | claude -p "Review this diff for:
1. Bugs (especially race conditions, off-by-one)
2. Security issues (XSS, SQL injection, secrets)
3. Performance issues (O(n^2), repeated fetches)
4. Style / consistency with codebase
5. Missing tests

Be specific. Reference line numbers. Suggest fixes.
Output format: Markdown with sections per category." --max-turns 3 --model sonnet --output-format text
```

---

## Template 5: Auto-Generate Tests

```
Generate tests via Print Mode:

Project path: [절대 경로]
Module path: <src/file.ts>
Test framework: <jest|vitest|pytest|...>

Tasks:
1. Identify public functions in <file>
2. For each, write happy path + 2 edge cases
3. Use existing test patterns (look for *.test.* files)

zsh -i -c 'cd <PROJECT_PATH> && claude -p "Generate comprehensive tests for <MODULE>. Use existing <FRAMEWORK> patterns. Cover happy path + 2 edge cases per function. Match existing assertions style." --allowedTools "Read,Write" --max-turns 10'

After: Run tests, ensure they pass.
```

---

## Template 6: Refactor with Plan Mode

```
Refactor <MODULE> using plan mode:

1. Use interactive tmux mode
2. First send: '/plan <MODULE>를 <NEW_PATTERN>으로 리팩토링하는 단계별 계획을 짜줘'
3. Wait for plan
4. Review plan with user (Claude may ask questions)
5. Approve plan
6. Send: '계획대로 구현 시작해줘'
7. After implementation: send '/review' to Claude itself
8. Fix any review issues found
9. Cleanup: /exit
```

---

## Template 7: Debug Interactively

```
Debug this issue via interactive tmux:

Symptom: [what's broken]
When: [conditions to reproduce]
Expected: [what should happen]
Actual: [what actually happens]

Steps:
1. tmux new-session -d -s debug -x 140 -y 40
2. Launch Claude, send task:
   '이 버그를 재현하고 원인 찾아줘:
    증상: <symptom>
    재현 조건: <when>
    기대 동작: <expected>
    실제 동작: <actual>

   최소 비용으로 진단해줘. 가설 → 검증 순서로.'
3. Watch Claude's diagnostic progression
4. Send follow-ups as it investigates
5. When Claude identifies root cause, ask for fix
6. Verify fix works
7. /exit
```

---

## Template 8: Bulk PR Workflow

```
Process N PRs efficiently:

For each of these PRs:
- PR #123: <title>
- PR #124: <title>
- PR #125: <title>

For each:
1. gh pr checkout <num>
2. Run code review (Template 4)
3. Report findings

You can do these sequentially in print mode — most reviews are < 5 min each.
Total expected: ~15-20 min.

Or parallel via tmux (3 sessions) for ~5-7 min wall-clock.
```

---

## Customizing Templates

- **Replace `<...>`** with actual values
- **Add constraints** as bullets in the Constraints section
- **For minimax/OpenRouter proxy**, ensure `~/.zshrc` is sourced (use `zsh -i -c`)
- **Always set budget caps** when doing many tasks

## What Makes a Good Prompt

| Good | Bad |
|---|---|
| Specific (file, line, function) | Vague ("make it better") |
| Has expected outcome | Open-ended |
| Lists constraints | Implicit assumptions |
| References existing patterns | Generic best-practice |
| Bounded scope | "Refactor the whole codebase" |

If a template feels vague, that's a sign to clarify with the user **before** running it.
