# ULW Loop — The Ralph Loop

## What Is It?

The **ULW Loop** (a.k.a. **Ralph Loop**, `/ulw-loop`) is the self-referential iteration mechanism that powers oh-my-openagent's Ultrawork mode.

> "Self-referential loop. Doesn't stop until 100% done."

The loop makes sure that even if the agent goes off-track, restarts from idle, or gets confused, the system **pulls it back** to continue until the task is verifiably complete.

## How It Works

```
┌───────────────────────────────────────┐
│  Step 1: Read current state           │
│     - Plan from .omo/plans/*.md       │
│     - Notepad from .omo/notepads/     │
│     - Work-in-progress                │
└───────────────────────────────────────┘
        ↓
┌───────────────────────────────────────┐
│  Step 2: Plan next iteration          │
│     - What's left?                    │
│     - What needs adjustment?          │
└───────────────────────────────────────┘
        ↓
┌───────────────────────────────────────┐
│  Step 3: Execute via tools/subagents  │
│     - task(category="...")            │
│     - call_omo_agent(subagent_type=.) │
└───────────────────────────────────────┘
        ↓
┌───────────────────────────────────────┐
│  Step 4: Verify against criteria      │
│     - Tests pass?                     │
│     - Code review OK?                 │
│     - Definition of Done met?         │
└───────────────────────────────────────┘
        ↓
       Done?
        ├── YES → Exit with report
        └── NO → Go back to Step 1
```

## Two Enforcement Mechanisms

### 1. ULW Loop (Self-Referential)

The agent itself drives the loop by re-evaluating state each iteration:

```python
# Pseudo-code (logical)
def ulw_loop(task):
    while not is_done(task):
        state = read_state()
        plan = next_iteration(state)
        execute(plan)
        verify()
```

### 2. Todo Enforcer (System-Level Safety Net)

If Sisyphus goes idle without reporting completion, **Todo Enforcer** hooks fire and yank the agent back:

> "Agent goes idle? System yanks it back. Your task gets done, period."

This is the safety net — even if the LLM tries to declare "I'm done" prematurely, the system continues checking against criteria.

## When Activated

- **Implicitly**: When you type `ulw <task>` or `ultrawork <task>` in Sisyphus
- **Explicitly**: `/ulw-loop` slash command
- **Configuration**: `ulw_loop.enabled: true` in `oh-my-openagent.json`

## Definition of Done

The loop verifies against a definition of done. Common criteria:

- [ ] All tasks in plan completed
- [ ] Code compiles / lints pass
- [ ] Tests pass (unit + integration)
- [ ] No regressions in existing functionality
- [ ] Documentation updated
- [ ] Git commits made with descriptive messages

> 💡 **Tip**: A vague task ("make this better") has no clear DoD. The loop will run forever (or hit budget cap). Make tasks specific.

## Loop Limits & Escape Hatches

Even ULW has limits. The loop will exit if:

| Condition | Action |
|---|---|
| All criteria met | Exit with success |
| `max_iterations` reached | Exit with partial report |
| `max_cost_usd` reached | Exit with budget message |
| Unrecoverable error | Exit with error |
| User interrupts (`Ctrl+C`) | Pause + prompt |

### Default Limits

```jsonc
{
  "ulw_loop": {
    "enabled": true,
    "max_iterations": 50,
    "max_cost_usd": 5.0,
    "max_wall_clock_hours": 2
  }
}
```

> ⚠️ These are typical defaults — check `oh-my-openagent.json` for actual values.

## Cost Considerations

ULW loop can be **expensive** because:

- Multiple agents active per iteration
- Background explore/librarian agents
- Wisdom accumulation grows context
- Verification steps add overhead

**Cost optimization**:
- Use `category="quick"` for trivial delegated tasks
- Run explore/librarian in parallel (one query, multiple sources)
- Stop the loop early if no progress after N iterations
- Use cheaper models (haiku/flash) for verification

## Evidence Audit

`.omo/ulw-loop/` directory contains the audit trail:

```
.omo/ulw-loop/
├── logs/
│   ├── 2026-07-10_iteration-1.log
│   ├── 2026-07-10_iteration-2.log
│   └── ...
├── evidence/
│   ├── test-results.json
│   ├── lint-output.txt
│   └── ...
└── state.json
```

This is what makes it **durable multi-goal orchestration** — you can resume after interruption, audit what was done, and debug failures.

## Wisdom Accumulation Across Loops

Each iteration adds to the notepad:

```markdown
<!-- .omo/notepads/my-plan/learnings.md -->

## Conventions
- Always run `make test` before committing
- Use `feat:` prefix for new features

## Successes
- Using delegate_task(category="quick") for typo fixes saved ~$0.20
- Parallel explore + librarian cut research time by 60%

## Failures
- Don't trust Sisyphus when it says "I think it's done" without verification
- Postgres migrations need explicit transaction wrappers

## Gotchas
- The codebase uses `mod.ts` convention, not `index.ts`
- Watch out for circular deps in `src/services/`
```

Passed forward to **every subsequent subagent** in the loop.

## Breaking Out of the Loop

Sometimes ULW is **wrong** for the situation. Escape routes:

| Trigger | What happens |
|---|---|
| `cancel` keyword | Gracefully exit, save state |
| `pause` keyword | Pause for human intervention |
| User message during loop | Inject context, resume |
| Ctrl+C | Hard interrupt + state save |

## Common Pitfalls

1. **Vague task → infinite loop** — Always give specific DoD
2. **Cost runaway** — Set `max_cost_usd` defensively
3. **Loop pulls back too aggressively** — Sometimes you want to pause; use `pause` keyword
4. **Not reading evidence audit** — After long loops, the audit trail tells you what really happened

## Verification Checklist

Before relying on ULW loop output:

- [ ] Definition of Done was specific (not vague)
- [ ] `.omo/ulw-loop/logs/` shows evidence of verification
- [ ] Wisdom notepad has documented learning
- [ ] Total cost is within your budget
- [ ] Git history shows actual file changes
