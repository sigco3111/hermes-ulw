# The 11 Agents — Deep Dive

> **Source**: code-yeongyu/oh-my-openagent `docs/guide/orchestration.md`

## Agent Inventory (Current as of v0.3.0)

The system has **11 built-in agents**:

- **Primary**: `sisyphus`, `hephaestus`, `prometheus`, `atlas`
- **Subagent**: `oracle`, `librarian`, `explore`, `multimodal-looker`, `metis`, `momus`, `sisyphus-junior`

---

## Primary Agents

### Sisyphus (Primary Worker)

- **Role**: Main coding worker. The default agent activated by `ulw` keyword.
- **Default Model**: `claude-opus-4-7` / `kimi-k2.5` / `gpt-5.5` / `glm-5` (fallback stack)
- **Best for**: General complex tasks, "just do it" scenarios with `ulw` keyword
- **Approach**: Keyword-activated ultrawork mode; uses Prometheus plans if available, otherwise autonomous exploration
- **Recommendation**: **For 90% of users** — use `ulw` keyword in Sisyphus

### Hephaestus (Autonomous Deep Worker)

- **Role**: Autonomous deep-mode worker — AmpCode-style exploration and execution.
- **Default Model**: `gpt-5.5` (medium)
- **Best for**: Complex architectural work, deep reasoning, problems benefiting from GPT-5.5's training
- **Approach**: Heavy use of `explore` / `librarian` agents, self-plans during execution
- **Recommendation**: **For power users** — switch to Hephaestus when you specifically need GPT-5.5's reasoning style

### Prometheus (Planner)

- **Role**: Generates implementation plans. Interview Mode.
- **Default Model**: `claude-opus-4-7` / `gpt-5.5` / `glm-5`
- **Best for**: Multi-step complex tasks that need upfront planning
- **Approach**: Interview user → research codebase → generate `.omo/plans/*.md`
- **Constraint**: Constrained to `.omo/*.md` writes (via `prometheus-md-only` hook)
- **Usage**: `Tab` → select Prometheus, OR `@plan` keyword in Sisyphus

### Atlas (Conductor / Orchestrator)

- **Role**: Reads plans, delegates tasks to subagents, verifies results.
- **Default Model**: `claude-sonnet-4-6` / `kimi-k2.6` / `gpt-5.5` / `minimax-m3` / `minimax-m2.7`
- **Best for**: Executing multi-step plans efficiently
- **Activates**: Automatically when you run `/start-work`
- **6 Steps**:
  1. Read Plan
  2. Analyze Tasks
  3. Accumulate Wisdom
  4. Delegate Tasks
  5. Verify Results
  6. Final Report

---

## Subagents

### Oracle (Architecture Advisor)

- **Role**: Read-only architecture advisor. Cannot edit code.
- **Default Model**: `gpt-5.5` / `gemini-3.1-pro` / `claude-opus-4-7` / `glm-5`
- **Best for**: Architecture decisions, design reviews, "what's the right approach?"
- **Hard-rejected** from Team Mode (read-only)

### Librarian (Documentation/Dependencies)

- **Role**: Looks up external documentation and dependencies.
- **Best for**: "How do I use library X?" "What's the API for Y?"
- **Run in background** to save time

### Explore (Codebase Pattern Finder)

- **Role**: Searches codebase for existing patterns, conventions, anti-patterns.
- **Best for**: "Find code that does X" / "What conventions does this project use?"
- **Run in background** to save time

### Multimodal-Looker (Image/PDF Analysis)

- **Role**: Analyzes images and PDFs (sketches, diagrams, screenshots).
- **Best for**: UI mocks, screenshots of bugs, design references
- **Hard-rejected** from Team Mode

### Metis (Pre-Planning Consultant)

- **Role**: Catches ambiguities **before** Prometheus planning.
- **Best for**: Vague requirements ("add auth" — what kind?)
- **Hard-rejected** from Team Mode

### Momus (Verification Reviewer)

- **Role**: Reviews plans for completeness. Returns `OKAY` or `REJECT`.
- **Default Model**: `gpt-5.5` / `claude-opus-4-7` / `gemini-3.1-pro` / `glm-5`
- **Best for**: Catching incomplete plans before execution wastes time
- **Hard-rejected** from Team Mode

### Sisyphus-Junior (Task Executor)

- **Role**: Like Sisyphus but for individual delegated tasks. Used by Atlas.
- **Default Model**: `claude-sonnet-4-6` / `kimi-k2.6` / `gpt-5.5` / `minimax-m3` / `minimax-m2.7`
- **Best for**: Executing specific delegated tasks from Atlas
- **Eligible** for Team Mode

---

## How Agents Communicate

```
User intent
   ↓
Sisyphus (or Hephaestus)
   ↓
[Prometheus plan?]
   ├── No → Autonomous exploration via Explore/Librarian
   └── Yes → Read plan
              ↓
           [Momus verify?]
              ├── REJECT → Prometheus iterates
              └── OKAY → Continue
                       ↓
                    Atlas executes
                       ↓
                    Delegates to subagents
                       ↓
                    Returns results
                       ↓
                    Wisdom accumulated
                       ↓
                    Next iteration or report complete
```

## Selection Heuristics

Use this table when deciding which agent activates:

| Situation | Recommended Agent | How |
|---|---|---|
| Simple fix | (just prompt, no agent change) | Direct prompt |
| `ulw <task>` keyword | Sisyphus | Type `ulw` |
| Need precise plan first | Prometheus + Atlas | `@plan` → `/start-work` |
| Deep architectural work | Hephaestus + Sisyphus | Tab → Hephaestus, then `ulw <task>` |
| Just research | Explore + Librarian | Background agents |
| Verify output | Momus | Auto in plan flow |

## Selection by Task Type

| Task Type | Best Agent Combination |
|---|---|
| Bug fix (single file) | Just prompt (no special agent) |
| Bug fix (multi-file) | Sisyphus + `ulw` |
| Refactor (scoped) | Sisyphus + `ulw` |
| Refactor (architectural) | Hephaestus + Sisyphus + `ulw` |
| New feature (well-defined) | Prometheus plan → Sisyphus execute |
| New feature (vague) | Metis consult → Prometheus plan → Sisyphus execute |
| Code search | Sisyphus + Explore (background) |
| Documentation lookup | Sisyphus + Librarian (background) |
| Architecture review | Oracle (read-only) |
| UI from mock | Sisyphus + Multimodal-Looker |

## Agent Constraints

Each agent has constraints enforced by hooks:

- **Oracle**: Read-only (no write/edit/patch/delegate)
- **Prometheus**: Markdown-only in `.omo/`
- **Explore**: Read-only
- **Librarian**: Read-only
- **Multimodal-Looker**: Read-only
- **Metis**: Read-only (pre-planning)
- **Momus**: Reads plans, returns verdict (no edits)

## Configuration

```jsonc
// oh-my-openagent.json
{
  "sisyphus_agent": {
    "disabled": false, // Enable Atlas orchestration (default: false)
    "planner_enabled": true, // Enable Prometheus (default: true)
    "replace_plan": true, // Replace default plan agent with Prometheus (default: true)
  },
  "disabled_hooks": [
    // "start-work",             // Disable execution trigger
    // "prometheus-md-only",     // Remove Prometheus write restrictions (not recommended)
  ]
}
```

## Common Pitfalls

1. **Picking the wrong agent for the task** — Hephaestus vs Sisyphus matters because of model reasoning style.
2. **Switching agents mid-task** — causes context loss; pick first.
3. **Oracle expects tasks it can't do** — Oracle is read-only; ask it for advice only.
4. **Momus confused with retry mechanism** — Momus only verifies plans, doesn't iterate runs.
5. **Background agents without monitoring** — can stall; check `.omo/notepads/` periodically.

## Quick Reference

| Want | Type | Agent |
|---|---|---|
| Quick fix | `fix this bug` | Sisyphus |
| Complex figure-it-out | `ulw <task>` | Sisyphus + 11 agents |
| Plan first | `@plan <task>` | Prometheus |
| Execute plan | `/start-work` | Atlas + workers |
| Just research | `find pattern X` | Sisyphus + Explore |
| Deep reasoning | Switch to Hephaestus + `ulw` | Hephaestus |
