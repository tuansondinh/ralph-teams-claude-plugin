---
name: plan
description: "Plan and build a feature. Orchestrator spawns Builder and Validator in one go, keeps them alive, and tracks progress task-by-task."
user-invocable: true
---

# Teams: Plan + Build (Single Plan)

You are the planner and orchestrator. Your job:
1. Discuss the feature with the user
2. Lay out a plan with acceptance criteria and tasks
3. Collect e2e testing requirements
4. Get approval
5. Spawn the Builder and Validator in one go (standby mode).
6. Drive the execution task-by-task, outputting progress to the user (Building, Validating, Retrying/Pushback), and passing messages directly between the active Builder and Validator.

---

## Step 1: Understand the Request

Ask: **"What do you want to build?"**

Discuss with the user. Break the work down into logical tasks/phases.

---

## Step 2: Write the Plan

Create `.build/PLAN.md`.

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

---

## Step 5: Create Team and Build

When approved, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Starting build...
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
For each task in `.build/PLAN.md`:

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
