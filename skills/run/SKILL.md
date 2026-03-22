---
name: run
description: "Resume building a single Teams plan. Orchestrator spawns Builder and Validator in one go, keeps them alive, and tracks progress task-by-task."
user-invocable: true
---

# Teams: Run (Resume Build)

You are the orchestrator. Your job: resume an existing build by spawning the Builder and Validator (in one go), and driving the execution task-by-task, outputting progress to the user (Building, Validating, Retrying/Pushback).

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

**A. Spawn the Team in one go:**
1. Use `Task` tool (`subagent_type: "teams:teams-builder"`) with prompt: 
   *"You are the Builder. Acknowledge and stand by. I will assign you tasks one by one."*
   Extract `BUILDER_TASK_ID`.
2. Use `Task` tool (`subagent_type: "teams:teams-validator"`) with prompt: 
   *"You are the Validator. Acknowledge and stand by. I will ask you to review tasks one by one."*
   Extract `VALIDATOR_TASK_ID`.

**B. Execute Task by Task:**
For each incomplete task in `.build/PLAN.md`:

1. **Building:**
   Print to user: `► Task [N]: Building...`
   Send to Builder (`task_id: BUILDER_TASK_ID`): 
   *"Implement Task [N]: [Description]. Here is the full plan context: [Plan]. When done, commit and return the SHA."*
   Wait for Builder.

2. **Validating:**
   Print to user: `► Task [N]: Validating...`
   Send to Validator (`task_id: VALIDATOR_TASK_ID`):
   *"Builder finished Task [N]. Commit SHA: [SHA]. Please review against the plan criteria and return PASS or FAIL with feedback."*
   Wait for Validator.

3. **Pushbacks / Retries (if FAIL):**
   If Validator returns FAIL:
   Print to user: `⟲ Task [N]: Pushback received. Retrying...`
   Send to Builder (`task_id: BUILDER_TASK_ID`):
   *"Validator returned FAIL with this feedback: [Validator Feedback]. Please fix, commit, and return new SHA."*
   Wait for Builder, then loop back to **Validating** (Max 2 pushbacks).

4. **Completion:**
   If Validator returns PASS (or max retries reached):
   Print to user: `✓ Task [N]: Complete!`
   Update `.build/PLAN.md` to check off the task.
   Move to the next task.

When all tasks are done, print a final success summary to the user.
