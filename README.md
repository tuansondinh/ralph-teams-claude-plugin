# ralph-teams

A Claude Code plugin that plans and builds features using a native agent team (a builder and validator). Supports a single-plan mode for most features and a phased loop mode for very large ones.

## How it works

1. `/teams-plan` — discuss what you want to build, Claude creates a plan with tasks and acceptance criteria in `.build/PLAN.md`, optionally runs an AI review of the plan, you approve, execution starts automatically
2. Execution: a native **Agent Team** is created. The **Builder** implements one task at a time, commits, and messages the **Validator** directly to request a review. The **Validator** reviews the commit against the acceptance criteria and replies with `PASS` or `FAIL`. On `FAIL`, the Builder fixes and re-submits.
3. E2E testing requirements and task status are tracked in `.build/PLAN.md`.

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
    "@ralph-teams-claude-plugin": true
  }
}
```

## Skills

| Command | Description |
|---------|-------------|
| `/teams-plan` | Discuss → plan → collect E2E requirements → optional AI review → approve → execute with a Builder + Validator team |
| `/teams-run` | Resume an existing single plan from where it left off |

## Output

Progress is shown as a live task board, reprinted after each status change:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  2 of 4 tasks complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: Project Setup          [done]
  ✓  Task 2: Auth System            [done]
  ►  Task 3: API Routes             [building...]
  ○  Task 4: Frontend               [pending]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
► Task 3: Building...
```

Status symbols: `✓` done · `►` active · `⟲` retrying · `○` pending
