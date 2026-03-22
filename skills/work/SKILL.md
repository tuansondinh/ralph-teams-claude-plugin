---
name: work
description: "Execute an existing Teams build plan. Use when user says '/teams:work', 'run the teams build', 'execute the plan', 'resume the build', or wants to re-run or continue an existing .build/PLAN.md."
user-invocable: true
---

# Teams: Work (Resume Build)

You are the Team Lead. Your job: find an existing `.build/PLAN.md`, resume the build, and spawn a team-lead agent to orchestrate completion.

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

## Step 2: Create Team and Spawn Lead

Print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Resuming build...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Then:

1. **Create a team** using `TeamCreate`:
   - `team_name`: derive from the plan title the same way as in `/teams:plan`, e.g., `"teams-auth-system"` (slugified, lowercase, hyphens)
   - `description`: "[Feature name] — agent team execution"

   (If a team with this name already exists from a prior run, `TeamCreate` will reuse it or error — adjust as needed)

2. **Spawn the Team Lead agent** using `Agent`:
   - `team_name`: the team name
   - `name`: "team-lead"
   - `subagent_type: "teams:teams-lead"`
   - `mode: "bypassPermissions"`
   - `prompt`:
   ```
   Resume the build. Read .build/PLAN.md and continue from where the last run left off.

   Current directory: [working directory]
   Plan file: .build/PLAN.md

   Your job:
   1. Read .build/PLAN.md
   2. For each phase:
      - If status is "done", skip it
      - If status is "pending" or "partial", rebuild it
   3. For each phase to build:
      - Create a task with TaskCreate
      - Spawn a builder agent
      - Spawn a validator
      - Manage retry if needed
      - Update task status
      - Update .build/PLAN.md
      - Message progress
   4. After all phases, summarize results

   Go.
   ```

3. **Return immediately** — the team-lead agent continues autonomously.

Print:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Team resumed! Team Lead is building...
  Check messages below for progress updates.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Notes

- If all phases are done, the team lead will report completion
- If some phases need rebuilding, they will be re-executed
- Check `.build/PLAN.md` for real-time progress
