---
name: run
description: "Resume building a single Teams plan. Orchestrator spawns Builder and Validator together; Builder tackles tasks one by one and validates directly."
user-invocable: true
---

# Teams: Run (Resume Build)

You are the orchestrator. Your job: resume an existing build by spawning the Validator on standby, then spawning the Builder to finish the tasks.

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

1. **Spawn the Validator on standby:**
   Use the `Task` tool (or `TaskCreate`) with `subagent_type: "teams:teams-validator"`.
   Prompt: "You are the Validator for this feature. Please acknowledge and stand by. I will pass your task_id to the Builder, who will contact you to review tasks one by one."
   **Wait for the Validator to return, and extract its `task_id`.**

2. **Spawn the Builder:**
   Use the `Task` tool with `subagent_type: "teams:teams-builder"`.
   Prompt:
   ```
   === PLAN SPEC ===
   [Copy the full Plan here: Tasks, Acceptance Criteria, E2E Requirements]

   === VALIDATOR TASK ID ===
   [Insert the Validator's task_id here]
   ```

3. **Wait for the Builder to complete.** 
   (The Builder will loop through the remaining tasks one by one, using the Validator's `task_id` to directly request reviews for each task until everything is finished).

4. **Complete:** 
   When the Builder returns its final report, the plan is complete. Print the final status to the user.
