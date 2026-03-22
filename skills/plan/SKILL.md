---
name: plan
description: "Plan and build a feature. Orchestrator spawns Builder and Validator together; Builder tackles tasks one by one and validates directly."
user-invocable: true
---

# Teams: Plan + Build (Single Plan)

You are the planner and orchestrator. Your job:
1. Discuss the feature with the user
2. Lay out a plan with acceptance criteria and tasks
3. Collect e2e testing requirements
4. Get approval
5. Spawn the Validator (on standby) and the Builder. The Builder will execute tasks one by one, communicating directly with the Validator.

---

## Step 1: Understand the Request

Ask: **"What do you want to build?"**

Discuss with the user. If unclear, ask 2-3 clarifying questions. Break the work down into logical tasks/phases.

---

## Step 2: Write the Plan

Create `.build/PLAN.md`. Lay out the acceptance criteria and the tasks.

```markdown
# Teams Plan: [Feature Name]

Generated: [date]
Status: approved
Mode: single

## Summary
[What we're building - 2-4 sentences]

## Tasks
1. [ ] Task 1: [Description]
2. [ ] Task 2: [Description]
3. [ ] Task 3: [Description]

## Acceptance Criteria
- [Criterion 1]
- [Criterion 2]
- Tests pass
- E2E tests pass

## E2E Testing Requirements
[To be collected from user]
```

---

## Step 3: Collect E2E Requirements

**Ask the user:**

> Before we build, I need to know what we'll test:
>
> 1. **E2E scenarios** - what user workflows should the validator test?
> 2. **Environment setup** - what do we need? Test accounts? API keys? Database setup?
> 3. **Test data** - any specific data the validator needs?
> 4. **Tool preference** - Playwright or Maestro for e2e? (default: Playwright)
> 5. **.env file** - provide a `.env.example` or list what variables we need for testing

Document the user's answers in `.build/PLAN.md` under `## E2E Testing Requirements`.

---

## Step 4: Show Plan and Get Approval

Display `.build/PLAN.md` and ask:

> **Plan and e2e requirements look good?**
>
> Reply with `yes` to start, or tell me what to change.

Update and re-show if needed.

---

## Step 5: Create Team and Build

When approved, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Starting build...
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
   (The Builder will loop through the tasks one by one, using the Validator's `task_id` to directly request reviews for each task until everything is finished).

4. **Complete:** 
   When the Builder returns its final report, the plan is complete. Print the final status to the user.
