---
name: teams-run
description: "Resume building a single Teams plan. Orchestrator spawns the native Agent Team and monitors progress task-by-task."
user-invocable: true
---

# Teams: Run (Resume Build)

You are the orchestrator and team lead. Your job: resume an existing build by spawning the Builder and Validator as a native Agent Team, and driving the execution task-by-task by monitoring the shared task list, outputting progress to the user (Building, Validating, Retrying/Pushback).

---

## Step 1: Find the Plan

```bash
cat .build/PLAN.md
```

If not found, tell user:
> `.build/PLAN.md` not found. Use `/teams-plan` to create a plan first.

---

## Step 2: Build

When approved, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Resuming build...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**A. Spawn the Team:**
1. Instruct the system to create an agent team containing a `teams-builder` and a `teams-validator`.
2. Add all incomplete tasks from the plan to the **shared task list** as "pending".
3. Assign the first incomplete task to the Builder to resume execution.

**B. Monitor Progress (Passive Observer):**
The Builder and Validator communicate directly via the `message` tool — the orchestrator cannot see those exchanges. You can only observe the **shared task list**. Watch for task status changes and reprint the task board each time one occurs.

Task board format:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  [N of M tasks complete]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: Project Setup          [done]
  ►  Task 2: Auth System            [in progress]
  ○  Task 3: API Routes             [pending]
  ○  Task 4: Frontend               [pending]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Status symbols (derived from shared task list only):
- `✓` — task status is "completed"
- `►` — task status is "in progress" (claimed by Builder)
- `○` — task status is "pending"

When all tasks are completed, ask the teammates to shut down, run team cleanup, and print a final success summary:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  All [M] tasks complete!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: ...
  ✓  Task 2: ...
  ...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
