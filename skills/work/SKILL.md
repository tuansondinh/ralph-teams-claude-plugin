---
name: work
description: "Execute an existing Teams build plan. Use when user says '/teams:work', 'run the teams build', 'execute the plan', 'resume the build', or wants to re-run or continue an existing .build/PLAN.md."
user-invocable: true
---

# Teams: Work (Resume Build)

You are a skill that resumes builds. Your job:
1. Find the existing `.build/PLAN.md`
2. Create a team and spawn the team-lead teammate
3. Display progress as the team works
4. Return when done

---

## Step 1: Find the Plan

```bash
cat .build/PLAN.md
```

If the file doesn't exist, tell the user:
> `.build/PLAN.md` not found. Use `/teams:plan` to create a plan first.

If it exists, extract:
- The feature name from the title
- Total phase count
- Current phases: which are done, pending, partial, failed?

---

## Step 2: Create Team and Spawn Team Lead

Print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Resuming build...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 1. Create the team

Use `TeamCreate`:
- `team_name`: derived from plan title, e.g., `"teams-auth-system"` (slugified, lowercase, hyphens)
- `description`: "[Feature name] — agent team execution"

### 2. Spawn the team-lead teammate

Use `Agent` with:
- `team_name`: the team name from step 1
- `name`: "team-lead"
- `subagent_type: "teams:teams-lead"`
- `mode: "bypassPermissions"`

Prompt:
```
Resume the build from .build/PLAN.md.

Your job:
1. Read .build/PLAN.md
2. For each phase that is NOT "done" status, spawn builder and validator teammates
3. Update the plan and tasks after each phase
4. When all incomplete phases are built, report completion

Go!
```

### 3. Display progress

The team-lead teammate now runs independently and messages progress. Display its messages and task updates to the user in real-time.

Show phase completions, builder commits, validator verdicts, and final summary as they happen.
