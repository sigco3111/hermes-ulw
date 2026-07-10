# ULW Prompt Templates

Copy-paste these into OpenCode (after `bunx oh-my-openagent install`).

---

## Template 1: Simple ULW Trigger (Lazy Mode)

```
ulw <one-line task description>
```

Examples:
- `ulw fix the failing tests in src/auth/`
- `ulw add JWT authentication following our patterns`
- `ulw refactor the auth module to use role-based access control`
- `ulw create a new CLI command for deployments`

Best for: Tasks where you trust the agent to figure out context.

---

## Template 2: ULW with Constraints

```
ulw <task>

Constraints:
- Match existing <style/conventions>
- Files involved: <glob>
- Don't change unrelated files
- Definition of Done: <specific measurable criteria>
- Cost cap: $<X>
```

Example:
```
ulw migrate user.service.ts from CommonJS to ESM

Constraints:
- Match existing import style (named imports, no default exports)
- Files involved: src/users/**/*.ts
- Don't change external API signatures
- Definition of Done: all imports resolve, all existing tests pass
- Cost cap: $1.00
```

---

## Template 3: Precise Plan First (Prometheus Interview)

```
@plan <detailed task description>

Context:
- Why this work is needed
- Constraints / non-negotiables
- Files / modules affected
- Acceptance criteria
```

Example:
```
@plan refactor the auth module to use role-based access control

Context:
- Current auth uses simple user/admin flags; we need granular roles (admin, editor, viewer)
- Existing JWT infrastructure stays — just add role claims
- 47 endpoints currently check auth; will need to migrate each
- Need backward compatibility for one release cycle

Constraints:
- No breaking changes to JWT format
- Roles defined in code (no DB schema change yet)
- Existing tests must pass

Acceptance Criteria:
- [ ] Role enum defined
- [ ] JWT includes roles claim
- [ ] Helper functions for role-checking
- [ ] Documentation updated
- [ ] All existing tests pass
```

After Prometheus generates the plan → run `/start-work` to execute.

---

## Template 4: Deep Architectural Reasoning (Hephaestus)

```
# 1. Press Tab, select Hephaestus agent
# 2. Then type:
ulw <architecturally complex task>
```

Examples:
- `ulw migrate from MongoDB to PostgreSQL with zero downtime`
- `ulw design event-sourced architecture for the order service`
- `ulw investigate the memory leak in the worker pool`

Best for: Tasks benefiting from GPT-5.5 deep reasoning style.

---

## Template 5: Team Mode (Parallel Multi-Agent)

```
# Team Mode needs opt-in: omo.json has "team_mode": true
ulw <task that benefits from parallel exploration>
```

Example scenario for Team Mode:
- Multi-faceted audit (5 hostile critics on same plan)
- Security research (3 hunters + 2 PoC engineers)

```
ulw audit the user service for security vulnerabilities
```

→ Triggers `security-research` profile: 3 hunters + 2 PoC engineers in parallel.

---

## Template 6: Background Research + Implementation

```
ulw <task that needs upfront research>

# Inside this prompt, you can hint to use background agents:
ulw add API endpoints for the new payment service. Run background librarian + explore first to understand existing service patterns.
```

→ OMO will spawn explore + librarian in background, then start implementation.

---

## Template 7: Multi-Goal Iteration

```
ulw <task>:
- Goal 1: <specific deliverable>
- Goal 2: <specific deliverable>
- Goal 3: <specific deliverable>

Definition of Done:
- Each goal has measurable acceptance criterion
- All goals verified before reporting done
```

ULW Loop runs until all goals meet their criteria.

---

## Template 8: Architecture Review (Oracle Only)

```
# Oracle is read-only — invoke via Sisyphus
ulw review the proposed architecture for the new notifications system.

# Specifically ask Oracle:
ask_oracle: should we use webhooks, SSE, or polling for the notifications system given:
- 10k concurrent users expected
- Sub-second delivery required
- Mobile + web clients

Constraints:
- Read-only review
- Recommendation with tradeoffs
```

→ Oracle (via Sisyphus) will give architecture advice without writing code.

---

## Template 9: Reset / Cancel Mid-Loop

If ULW loop is going wrong:

```
# Pause for human intervention
pause

# After intervening:
resume

# If you want to abandon the current loop:
cancel
```

---

## Template 10: Pattern for Specific Coding Tasks

### Bug fix
```
ulw fix the bug where <describe symptom>.
- Reproduction steps: ...
- Expected: ...
- Actual: ...
- Likely files: ...
```

### Refactor
```
ulw refactor <target> to <new pattern>.
- Reason: ...
- Constraints: ...
- Should not change: <list>
```

### New feature
```
ulw add <feature>.
- Use case: ...
- API surface: ...
- Should integrate with: <existing modules>
- Tests required: <yes/no>
```

### Documentation
```
ulw write docs for <target>.
- Audience: <developer | user>
- Format: <markdown | typedoc | JSDoc>
- Required sections: ...
```

---

## Customizing Templates

| Customization | When |
|---|---|
| Add `@plan` for upfront planning | When you need precise, verifiable execution |
| Switch to Hephaestus | When you need GPT-5.5 deep reasoning |
| Add `ulw` keyword | When agent should figure out context |
| Add explicit constraints | When there's risk of scope creep |
| Specify Definition of Done | When task has multiple success criteria |

## What Makes a Good ULW Prompt

| Good | Bad |
|---|---|
| Specific scope ("`ulw add OAuth to /api/auth`") | Vague scope ("make auth better") |
| Has Definition of Done | Open-ended |
| Mentions constraints | Implicit assumptions |
| References existing patterns ("follow our patterns") | Generic best-practice |
| Specifies cost cap | No budget awareness |

> 💡 **Tip**: Even ULW benefits from clarity. "Just do it mode" doesn't mean "do anything you want" — give a clear goal.
