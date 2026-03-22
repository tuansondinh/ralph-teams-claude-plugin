# ralph-teams

A Claude Code plugin that plans and builds features using an agent team (a builder and validator). By default, it plans and builds the entire feature in one shot. It also supports a "loop" mode for very large features, breaking them into phases where a fresh builder + validator team is spawned per phase.

## How it works

1. `/teams:plan` — discuss what you want to build, Claude creates a plan in `.build/PLAN.md`, you approve, execution starts automatically
2. Execution: **Builder** implements the plan → **Validator** reviews it → Validator pushes back directly if fixes are needed (max 2 times).
3. Status and e2e testing requirements are tracked in `.build/PLAN.md`.

## Install

Add to your `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "ralph-teams-claude-plugin": {
      "source": {
        "source": "github",
        "repo": "tuansondinh/ralph-teams-claude-plugin"
      }
    }
  },
  "enabledPlugins": {
    "teams@ralph-teams-claude-plugin": true
  }
}
```

## Skills

| Skill | Trigger | Description |
|-------|---------|-------------|
| `teams:plan` | `/teams:plan` | Discuss → plan → approve → execute as a single big team effort |
| `teams:run` | `/teams:run` | Resume an existing single plan |
| `teams:loop-plan` | `/teams:loop-plan` | Discuss → plan → approve → execute in loop phases |
| `teams:loop-run` | `/teams:loop-run` | Resume an existing phased plan |

## Phase sizing

Each phase should be large enough to use ~50–60% of a builder's context window. Think "implement the full auth system" not "add a login button". Phases are sequential — each builds on the previous.

## Output

Progress is shown visually as phases run:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Phase 2/4 — Auth Middleware
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Phase 1: Database Schema      [done]
  ►  Phase 2: Auth Middleware      [building...]
  ○  Phase 3: API Routes           [pending]
  ○  Phase 4: Frontend             [pending]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Final status per phase: `done` · `partial` · `failed`
