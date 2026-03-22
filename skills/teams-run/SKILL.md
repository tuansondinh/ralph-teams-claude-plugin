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

**B. Monitor and Route Progress to User:**
After each status change, reprint the full task board followed by the current status line. This replaces the previous output so the user always sees the complete picture.

Task board format:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  [N of M tasks complete]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: Project Setup          [done]
  ►  Task 2: Auth System            [building...]
  ○  Task 3: API Routes             [pending]
  ○  Task 4: Frontend               [pending]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Status symbols:
- `✓` — completed (Validator returned PASS)
- `►` — active (Builder is working or Validator is reviewing)
- `⟲` — retrying (Validator returned FAIL, Builder is fixing)
- `○` — pending

Status line appended below the board on each event:
- Builder starts a task → `► Task [N]: Building...`
- Builder messages Validator → `► Task [N]: Validating...`
- Validator returns FAIL → `⟲ Task [N]: Pushback received. Retrying...`
- Validator returns PASS → `✓ Task [N]: Complete!`

After each status change, also update the task checkboxes in `.build/PLAN.md` to reflect current state (check off completed tasks).

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
