---
name: hermes-ulw
description: "Use when the user mentions 'ULW', 'ultrawork', 'oh-my-opencode', 'oh-my-openagent', 'omo', or asks to delegate a complex coding task with the Ultrawork keyword (e.g. 'ulw fix the failing tests', 'ulw add JWT auth', 'ulw refactor this module'). Routes the request into the oh-my-openagent Ultrawork workflow on OpenCode (Prometheus planning → Sisyphus/Atlas execution → Momus verification → ULW loop until 100% done). Replaces 'lazy' manual prompting with the agent figuring out context, planning, delegating to specialized subagents (oracle, librarian, explore, multimodal-looker, metis, momus, sisyphus-junior), and not stopping until verified completion."
version: 0.3.0
author: hjshin (sigco3111)
license: MIT
platforms: [linux, macos]
metadata:
  hermes:
    tags: [OpenCode, oh-my-openagent, omo, Ultrawork, ULW, Multi-Agent, Orchestration, Hermes-CLI]
    related_skills: [claude-code]
---

# Hermes ULW — Ultrawork Orchestration Guide

## Overview

**ULW = "Ultrawork"**, a feature of **oh-my-openagent (a.k.a. omo, formerly oh-my-opencode)**, an OpenCode plugin. Typing the keyword `ulw` (or `ultrawork`) in front of a task triggers a multi-agent orchestration that:

1. Activates the planning layer (**Prometheus**)
2. Activates the verification layer (**Momus**)
3. Activates the conductor (**Atlas**, **Sisyphus**)
4. Delegates to specialized subagents (**Oracle**, **Librarian**, **Explore**, **Multimodal-Looker**, **Metis**, **Sisyphus-Junior**)
5. **Loops until 100% done** (`ulw-loop`, Todo Enforcer pulls the agent back if idle)
6. Verifies work before declaring completion

In OMO's own words: **"One word. Every agent activates. Doesn't stop until done."**

This skill is the Hermes Agent guide for invoking and orchestrating that workflow.

> **Food analogy**: A regular prompt is a single chef cooking one dish. ULW is the **whole kitchen brigade** — sous chef preps, grill cooks, pastry finishes, manager verifies — all coordinated, and nobody leaves until the dish is plated.

## When to Use

**Trigger this skill when the user says any of:**
- `ulw <task>` or `ultrawork <task>` keyword in the prompt
- "oh-my-opencode", "oh-my-openagent", "omo" + a coding task
- "just do it mode", "lazy mode", "에이전트한테 다 맡겨"
- "Multi-agent orchestration", "Agent 팀에게 시키고 싶어"
- "Don't stop until done", "100% done 될 때까지"

**Examples of triggering prompts:**
- "ulw fix the failing tests"
- "ulw add JWT authentication following our patterns"
- "ultrawork refactor the auth module"
- "omo한테 시켜", "이거 끝까지 알아서 해줘"

**Don't trigger for:**
- Single-line file edits (no keyword needed)
- Questions (just answer normally)
- Non-coding tasks (use general delegation)

## Decision Flow (When Ultrawork Is Appropriate)

```
Is it a quick fix or simple task?
├── YES → Just prompt normally, no ulw needed
└── NO  → Is explaining the full context tedious?
          ├── YES → Type "ulw" and let the agent figure it out
          └── NO  → Do you need precise, verifiable execution?
                    ├── YES → @plan (Prometheus), then /start-work
                    └── NO  → Just use "ulw"
```

(Adapted from OMO's orchestration guide)

## The 5 Modes in OMO

| Mode | Trigger | What it does |
|---|---|---|
| **`ulw` / `ultrawork`** | Type keyword | All agents activate, orchestration begins, loops until done |
| **`search`** | Search-only | Find patterns, no edits |
| **`analyze`** | Read-only | Deep investigation, no changes |
| **`team`** | v4.0, opt-in | Lead agent + up to 8 parallel members |
| **`hyperplan`** | Team mode | 5 hostile critics reviewing a plan before execution |

## The 11 Agents

### Planning Layer (Human + Prometheus)

| Agent | Role | Default Model |
|---|---|---|
| **Prometheus** | Planner — interviews user, generates `.omo/plans/*.md` | opus / gpt-5 / glm-5 |
| **Metis** | Consultant — catches ambiguities | varies |

### Verification Layer

| Agent | Role | Default Model |
|---|---|---|
| **Momus** | Reviewer — checks plans are complete, returns OKAY/REJECT | opus / gpt-5 / gemini |

### Execution Layer (Orchestrator)

| Agent | Role | Default Model |
|---|---|---|
| **Atlas** | Conductor — reads plan, delegates tasks, verifies results | sonnet / kimi / gpt-5 / minimax |

### Worker Layer (Specialized Agents)

| Agent | Role | Default Model |
|---|---|---|
| **Sisyphus** | Primary worker | opus / kimi / gpt-5 |
| **Hephaestus** | Autonomous deep worker (AmpCode-style) | gpt-5 (medium) |
| **Sisyphus-Junior** | Task executor | sonnet / kimi / gpt-5 |
| **Oracle** | Read-only architecture advisor | gpt-5 / gemini / opus |
| **Explore** | Codebase pattern finder | subagent |
| **Librarian** | Documentation/dependency lookup | subagent |
| **Multimodal-Looker** | Image/PDF analysis | multimodal |

## What Happens When You Type `ulw <task>`

1. **IntentGate** analyzes your true intent (vs literal interpretation)
2. **Sisyphus** (or **Hephaestus** if you switched to it) activates
3. **Todo Continuation** kicks in — if the agent goes idle, system yanks it back
4. **Prometheus** (if enabled) generates a plan file in `.omo/plans/`
5. **Momus** reviews the plan → if REJECT, Prometheus iterates
6. **Atlas** activates, reads the plan, starts delegating:
   - `task(category="deep" | "quick" | "visual-engineering" | "ultrabrain" | ...)`
   - `task(subagent_type="oracle")` for architecture advice
   - `call_omo_agent(subagent_type="explore")` for code patterns
   - `call_omo_agent(subagent_type="librarian")` for docs
7. **Wisdom accumulation** — learnings from each subagent passed forward
8. **ulw-loop** ensures iteration until verified-done
9. Final report back to you

## ulw-loop / Ralph Loop

`/ulw-loop` (or just typing `ulw`) triggers a **self-referential loop**:

```
Hephaestus reads current state
  → Plans next iteration
    → Executes via tools/subagents
      → Verifies against criteria
        → Not done? → Loop back to top
        → Done → Report and exit
```

**Todo Enforcer** is the safety net: if Sisyphus goes idle without reporting completion, the system pulls it back to continue.

## Common Patterns

### "I want to add a feature" (lazy mode)

```
ulw add JWT authentication to the API
```

→ OMO figures out: existing patterns, library choices, file structure, tests, all on its own.

### "I want to fix this bug" (lazy mode)

```
ulw fix the failing tests in src/auth/
```

→ OMO will investigate, plan, implement, test, verify.

### "I need a precise plan first" (precise mode)

```
@plan refactor the auth module to use role-based access control
```

→ Prometheus interviews you, generates a plan, Momus verifies, then `/start-work` to execute.

### "I want deep architectural reasoning" (Hephaestus mode)

```
# Switch agents: Tab → Hephaestus
ulw migrate from MongoDB to PostgreSQL with zero downtime
```

→ Hephaestus explores deeply first, then orchestrates execution.

## Comparison: ulw vs /start-work vs Just-Prompt

| Approach | When to Use |
|---|---|
| Just-prompt | Simple, well-scoped tasks |
| `ulw <task>` | Complex but you trust the agent to figure it out |
| `@plan` → `/start-work` | Complex + need precise, verifiable execution |
| Hephaestus agent | Need GPT-5.5 deep reasoning style |

## When NOT to Use ULW

ULW is overkill for:

- ❌ Single-line edits
- ❌ Renaming a variable
- ❌ Questions / explanations
- ❌ Tasks with very narrow scope ("change this constant")

For those, just type normally — no `ulw` keyword needed.

## Common Pitfalls

1. **Don't prefix simple tasks with `ulw`** — wastes resources; ULW triggers 11-agent orchestration, overkill for trivial work.
2. **`ulw` ≠ a magic wand** — it's a starting trigger. The agent still needs enough context to interpret your task.
3. **Hephaestus vs ulw in Sisyphus** — Hephaestus is for "AmpCode deep mode" autonomous exploration. For most tasks, ulw in Sisyphus is better.
4. **`/ulw-loop` doesn't verify quality** — it verifies completion. Quality verification is done by Momus at planning time.
5. **Wisdom accumulation carries forward** — bad patterns get reinforced unless you correct them explicitly.

## Verification Checklist

After `ulw` task completes:

- [ ] All agents in the planning + execution + verification layers actually activated (check logs)
- [ ] Plan file in `.omo/plans/` (if Prometheus used)
- [ ] No stuck subagents
- [ ] Wisdom accumulation propagated to subsequent agents
- [ ] Final verification criteria met (mentioned in plan)

## Setup: Install oh-my-openagent

```bash
# Ultimate Edition (OpenCode + OMO)
bunx oh-my-openagent install

# Light Edition (Codex CLI)
npx lazycodex-ai install
```

The Ultimate Edition adds:

- 11 agents (including ulw-loop, ultrawork, Team Mode)
- 54+ lifecycle hooks
- 5 built-in MCPs (Exa, Context7, Grep.app)
- Hash-anchored edit tool
- All slash commands

## Install This Skill into Hermes Agent

This skill (`hermes-ulw`) is designed to be loaded by Hermes Agent. Install it as a **user-local skill** in the active Hermes profile:

```bash
# Clone this repo
git clone https://github.com/sigco3111/hermes-ulw.git /tmp/hermes-ulw

# Copy the SKILL.md into Hermes's skill directory
mkdir -p ~/.hermes/skills/autonomous-ai-agents/hermes-ulw
cp /tmp/hermes-ulw/SKILL.md ~/.hermes/skills/autonomous-ai-agents/hermes-ulw/SKILL.md
# Optionally copy the references/ directory for linked files:
cp -r /tmp/hermes-ulw/references ~/.hermes/skills/autonomous-ai-agents/hermes-ulw/

# Start a NEW Hermes session (skill loader caches at session start)
```

> 💡 In the new session, the auto-trigger description will match whenever you mention `ulw`, `ultrawork`, `omo`, `oh-my-opencode`, `oh-my-openagent`, or related keywords — and this SKILL.md will be loaded automatically.

To install from Hermes Agent itself (in-session):

```
# Ask Hermes to install (it will use skill_manage automatically):
"hermes-ulw 스킬 설치해줘" → (uses skill_manage action='create' from URL)
```

> ⚠️ The skill loader caches at session start. After installation, **start a new session** for the new skill to be visible.

## References

- `references/orchestration.md` — Full architecture of planning/execution/worker layers
- `references/agents.md` — All 11 agents and their roles in detail
- `references/ulw-loop.md` — The Ralph Loop / ULW loop mechanics
- `references/configuration.md` — `oh-my-openagent.json` settings
- `templates/ulw-prompt-template.md` — Copy-paste prompt templates for common ulw workflows

## One-Line Pitch

> **Hermes ULW = the oh-my-openagent Ultrawork keyword, brought into Hermes Agent — type `ulw <task>` and the agent figures out planning, execution, and verification on its own, looping until the work is verifiably done.**
