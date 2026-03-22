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

**B. Monitor Progress and Keep Teammates Active:**
The Builder and Validator communicate directly via the `message` tool — the orchestrator cannot see those exchanges. You can only observe the **shared task list**. Watch for task status changes and reprint the task board each time one occurs.

**The user is not present. Do not pause, ask questions, or wait for input at any point during execution. Run all tasks to completion autonomously. If a task fails after the Validator's maximum pushbacks, log it as failed in `.build/PLAN.md`, then continue with the next task without stopping.**

**Watchdog — prevent idle teammates:**
If the shared task list has not changed after a reasonable amount of time (no task moving to in-progress or completed):
1. Check which task is currently in progress and which teammate should be active.
2. Use the `message` tool to ping that teammate directly:
   - If no task is in progress and pending tasks remain → ping the Builder: `"There are still pending tasks. Please claim the next task and continue."`
   - If a task is in progress with no recent update → ping the Builder: `"Are you still working on [task]? Please continue or let me know if you're blocked."`
3. If the teammate does not respond or progress still stalls, re-assign the task on the shared task list and ping again.

Task board format:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  [N of M tasks complete]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: Project Setup          [done]
  ◉  Task 2: Auth System            [validating...]
  ○  Task 3: API Routes             [pending]
  ○  Task 4: Frontend               [pending]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Status symbols (derived from shared task list only):
- `✓` — task status is "completed"
- `►` — task status is "in progress" (Builder is implementing)
- `◉` — task status is "validating" (Validator is reviewing)
- `✗` — task status is "failed" (Validator could not approve after max pushbacks)
- `○` — task status is "pending"

When all tasks are completed, shut down the team, run cleanup, and print a final summary:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  All [M] tasks complete!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: ...
  ✓  Task 2: ...
  ...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
