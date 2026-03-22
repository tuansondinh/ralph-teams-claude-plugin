# ralph-teams

A Claude Code plugin that plans and builds features using an agent team — a builder and validator loop per phase, coordinated by you as Team Lead.

## How it works

1. `/teams:plan` — discuss what you want to build, Claude creates a phased plan in `.build/PLAN.md`, you approve, execution starts automatically
2. Each phase: **Builder** implements → **Validator** reviews → Builder applies feedback if needed → next phase
3. All phases run to completion regardless of failures. Status tracked in `.build/PLAN.md`.

## Install

Add to your `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "ralph-teams": {
      "source": {
        "source": "github",
        "repo": "tuansondinh/ralph-teams"
      }
    }
  },
  "enabledPlugins": {
    "teams@ralph-teams": true
  }
}
```

## Skills

| Skill | Trigger | Description |
|-------|---------|-------------|
| `teams:plan` | `/teams:plan` | Discuss → plan → approve → execute |
| `teams:work` | `/teams:work` | Re-run or resume an existing plan |

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
