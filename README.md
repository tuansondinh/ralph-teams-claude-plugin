# ralph-teams

A Claude Code plugin that plans and builds features using sequential Sonnet builder subagents, automated E2E verification, and an Opus code review pass.

## Quick Start

```
/teams:plan
```

That's it. Describe what you want to build — Claude handles the rest.

---

## Quick Install

```bash
python3 -c "
import json, pathlib
f = pathlib.Path.home() / '.claude/settings.json'
s = json.loads(f.read_text()) if f.exists() else {}
s.setdefault('extraKnownMarketplaces', {})['ralph-teams-claude-plugin'] = {'source': {'source': 'github', 'repo': 'tuansondinh/ralph-teams-claude-plugin'}}
s.setdefault('enabledPlugins', {})['@ralph-teams-claude-plugin'] = True
f.write_text(json.dumps(s, indent=2))
print('Done — restart Claude Code')
"
```

Then restart Claude Code and run `/teams:plan`.

---

## How it works

```
/teams:plan   →  Discuss feature → write plan → approve
/teams:run    →  Build each task (one fresh Sonnet agent per task)
              →  Opus reviewer checks the full implementation
              →  Builder applies any blocking fixes
/teams:verify →  Walk through E2E scenarios manually with you
```

Each task runs in its own isolated subagent with a clean context window. Results are committed after each task so you can always resume with `/teams:run`.

---

## Commands

| Command | What it does |
|---------|-------------|
| `/teams:plan` | Discuss → plan → optional AI review → approve → build → Opus review → fixes |
| `/teams:run` | Resume an existing plan from where it left off |
| `/teams:verify` | Walk through manual E2E verification scenario by scenario |
| `/teams:loop-plan` | Plan and build a large feature in sequential phases |
| `/teams:loop-run` | Resume a phased plan |

---

## Progress output

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

`✓` done · `►` building · `✗` failed · `○` pending

---

## Output files

All build artifacts are written to `./ralph-teams/` in your project:

| File | Contents |
|------|----------|
| `ralph-teams/PLAN.md` | Tasks, acceptance criteria, verification scenarios |
| `ralph-teams/REVIEW.md` | Opus reviewer findings (blocking / non-blocking) |
| `ralph-teams/VERIFY.md` | Manual verification results |
