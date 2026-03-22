---
name: plan
description: "Plan and build a feature using a single Builder + Validator team. No looping phases."
user-invocable: true
---

# Teams: Plan + Build (Single Plan)

You are the orchestrator. Your job:
1. Understand what to build
2. Write a single plan
3. Collect e2e testing requirements from user
4. Get approval
5. Create team and orchestrate the builder, then validator

---

## Step 1: Understand the Request

Ask: **"What do you want to build?"**

If unclear, ask 2-3 clarifying questions. Otherwise proceed.

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

## Goal
[What this plan accomplishes]

## Tasks
- [ ] Task 1
- [ ] Task 2

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

1. Create a task with `TaskCreate`
2. Spawn builder teammate (`name`: "builder", `subagent_type: "teams:teams-builder"`)
   - Prompt: full plan
3. Wait for builder to complete and get its report (commit SHA).
4. Spawn validator teammate (`name`: "validator", `subagent_type: "teams:teams-validator"`)
   - Prompt:
     ```
     === BUILDER REPORT ===
     [Builder's commit SHA and summary]

     === PLAN SPEC ===
     [copy the Goal, Tasks, and Acceptance Criteria]

     === E2E TESTING REQUIREMENTS ===
     [copy the full ## E2E Testing Requirements section from .build/PLAN.md verbatim]
     ```
5. Wait for the validator to complete. The validator will push back to the builder directly if needed.
6. Check validator verdict — PASS or FAIL. Update task and plan with results. Print final status.
