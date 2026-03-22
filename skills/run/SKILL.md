---
name: run
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
> `.build/PLAN.md` not found. Use `/teams:plan` to create a plan first.

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

**B. Monitor and Route Progress to User:**
While the Builder and Validator are working, you act as the observer to keep the user informed.
1. When the Builder begins a task, print: `► Task [N]: Building...`
2. When the Builder uses the `message` tool to ask the Validator for a review, print: `► Task [N]: Validating...`
3. When the Validator uses the `message` tool to push back with a `FAIL`, print: `⟲ Task [N]: Pushback received. Retrying...`
4. When the Validator returns `PASS` and the task is marked completed on the shared task list, print: `✓ Task [N]: Complete!`
   - If there are remaining tasks, ensure the Builder claims the next task.

When all tasks on the shared task list are "completed", ask the teammates to shut down, run team cleanup, and print a final success summary to the user.
