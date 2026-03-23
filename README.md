# ralph-teams

A Claude Code plugin that plans and builds features using sequential Sonnet builder subagents with automated E2E verification, followed by an Opus code review pass.

## How it works

1. `/teams:plan` — discuss what you want to build, Claude creates a plan with tasks, acceptance criteria, and verification scenarios in `ralph-teams/PLAN.md`. Optionally runs an AI review of the plan. You approve, execution starts.
2. **Build:** For each task, a Sonnet builder subagent is spawned sequentially. Each builder implements the task, verifies it with Playwright (web) or Maestro (mobile), and commits.
3. **Review:** After all tasks complete, an Opus reviewer checks the full implementation against acceptance criteria. It can optionally use Multi-CLI (Ask-Codex) for a second opinion. Findings are written to `ralph-teams/REVIEW.md`.
4. **Fix:** If the review finds blocking issues, a builder subagent is spawned to apply fixes.
5. `/teams:verify` — walk through manual E2E verification of each scenario with the user.

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
| `/teams:plan` | Discuss → plan → optional AI review → approve → build (sequential Sonnet builders) → Opus review → apply fixes |
| `/teams:run` | Resume an existing plan from where it left off |
| `/teams:verify` | Walk through manual E2E verification scenario by scenario |

## Output

Progress is shown as a task board, reprinted after each task completes:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  2 of 4 tasks complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: Project Setup          [done]
  ✓  Task 2: Auth System            [done]
  ►  Task 3: API Routes             [building...]
  ○  Task 4: Frontend               [pending]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Status symbols: `✓` done · `►` building · `✗` failed · `○` pending
