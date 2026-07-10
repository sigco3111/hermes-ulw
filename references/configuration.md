# Configuration Reference

## oh-my-openagent.json

The main config file lives at the root of your project (or in `~/.config/opencode/oh-my-openagent.json` for user-level).

### Sisyphus & Planning

```jsonc
{
  "sisyphus_agent": {
    "disabled": false,           // Enable Atlas orchestration (default: false)
    "planner_enabled": true,     // Enable Prometheus (default: true)
    "replace_plan": true,        // Replace default plan agent with Prometheus (default: true),
  }
}
```

### Hooks

```jsonc
{
  "disabled_hooks": [
    // "start-work",             // Disable execution trigger
    // "prometheus-md-only",     // Remove Prometheus write restrictions (not recommended)
    // "todo-continuation",      // Disable Todo Enforcer (very risky)
    // "ulw-loop",               // Disable ulw loop
  ]
}
```

### ULW Loop Settings

```jsonc
{
  "ulw_loop": {
    "enabled": true,
    "max_iterations": 50,
    "max_cost_usd": 5.0,
    "max_wall_clock_hours": 2,
    "evidence_audit": true      // Save logs to .omo/ulw-loop/
  }
}
```

### Background Concurrency

```jsonc
{
  "background_task": {
    "defaultConcurrency": 4,
    "providerConcurrency": {
      "anthropic": 2,
      "openai": 2,
      "ollama": 1
    },
    "modelConcurrency": {
      "claude-opus-4-7": 2,
      "gpt-5.5": 4
    }
  }
}
```

### Category Defaults

```jsonc
{
  "delegate_task": {
    "defaults": {
      "visual-engineering": {
        "model": "claude-sonnet-4-6",
        "load_skills": ["frontend", "design-system"]
      },
      "deep": {
        "model": "claude-opus-4-7"
      },
      "quick": {
        "model": "kimi-k2.6"  // cheaper
      }
    }
  }
}
```

## MCP Server Configuration

OMO ships with built-in MCPs (websearch, context7 docs, grep_app GitHub search). They're runtime-injected, not shown in `opencode mcp list`.

To add custom MCPs:

```jsonc
{
  "mcp": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"],
      "enabled": true
    }
  }
}
```

## AGENTS.md — Project-Level Rules

Project rules auto-load into agent context at every prompt.

**Location**: `./AGENTS.md` (project), `~/.claude/CLAUDE.md` (user)

```markdown
# Project: MyApp

## Build & Test
- Build: `npm run build`
- Test: `npm test` (single command runs all)
- Lint: `npm run lint`

## Conventions
- TypeScript strict mode
- ESLint config: @typescript-eslint/recommended
- Tests required for new utilities (>60% coverage)

## Architecture
- Monorepo with pnpm workspaces
- Backend: Fastify + Postgres
- Frontend: React + Vite
```

For larger projects, modular rules:

```
.omo/rules/
├── build.md
├── code-style.md
├── testing.md
└── architecture.md
```

## Modifying Behavior Per-Project

Override defaults in `oh-my-openagent.json`:

```jsonc
{
  "agents": {
    "sisyphus": {
      "default_model": "minimax-m3"  // override default
    },
    "prometheus": {
      "enabled": true  // explicit
    }
  }
}
```

## Environment Variables

| Variable | Purpose |
|---|---|
| `ANTHROPIC_API_KEY` | Claude (Anthropic) auth |
| `OPENAI_API_KEY` | OpenAI / GPT auth |
| `OPENROUTER_API_KEY` | OpenRouter auth |
| `MINIMAX_API_KEY` | minimax auth |
| `ZAI_API_KEY` | Z.ai / GLM auth |
| `OMO_TELEMETRY` | `1` to enable telemetry |

## Common Config Issues

### Symptom: Agents fall back to wrong model

```bash
# Check what model Sisyphus is using
omo status
# or look in log/output
```

Fix: Check `delegate_task.defaults` and per-agent `default_model` overrides.

### Symptom: ULW loop runs forever

```jsonc
{
  "ulw_loop": {
    "max_iterations": 20,    // reduce from 50
    "max_cost_usd": 2.0      // hard cap
  }
}
```

### Symptom: Background agents serializing

Set explicit concurrency:

```jsonc
{
  "background_task": {
    "providerConcurrency": {
      "anthropic": 4
    }
  }
}
```

### Symptom: Plan agent missing

```jsonc
{
  "sisyphus_agent": {
    "planner_enabled": true,
    "replace_plan": true    // use Prometheus, not default
  }
}
```

## Verification Checklist

After config changes:

- [ ] `omo status` shows expected agent states
- [ ] `ulw <task>` triggers all expected agents
- [ ] Background concurrency behaves as configured
- [ ] Budget caps honored
- [ ] Telemetry working (if enabled)
