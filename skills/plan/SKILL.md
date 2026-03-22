---
name: plan
description: "Plan and build a feature with an agent team. Builder and validator work in parallel."
user-invocable: true
---

# Teams: Plan + Build

You are the orchestrator. Your job:
1. Understand what to build
2. Write a phased plan
3. Collect e2e testing requirements from user
4. Get approval
5. Create team and orchestrate builder + validator in parallel for each phase

---

## Step 1: Understand the Request

Ask: **"What do you want to build?"**

If unclear, ask 2–3 clarifying questions. Otherwise proceed.

---

## Step 2: Write the Plan

Create `.build/PLAN.md` with phases (~100k–120k tokens per phase).

```markdown
# Teams Plan: [Feature Name]

Generated: [date]
Status: approved

## Summary
[What we're building — 2–4 sentences]

## E2E Testing Requirements
[To be collected from user]

---

### Phase 1: [Name]
Status: pending
Goal: [What this phase accomplishes]

Tasks:
- [ ] Task 1
- [ ] Task 2

Acceptance Criteria:
- [Criterion 1]
- [Criterion 2]
- Tests pass
- E2E tests pass

---

### Phase 2: [Name]
...
```

---

## Step 3: Collect E2E Requirements

**Ask the user:**

> Before we build, I need to know what we'll test:
>
> 1. **E2E scenarios** — what user workflows should the validator test? (e.g., "user signs up, logs in, creates a project")
> 2. **Environment setup** — what do we need? Test accounts? API keys? Database setup?
> 3. **Test data** — any specific data the validator needs? (e.g., test user credentials)
> 4. **Tool preference** — Playwright or Maestro for e2e? (default: Playwright)
> 5. **.env file** — provide a `.env.example` or list what variables we need for testing

Document the user's answers in `.build/PLAN.md` under `## E2E Testing Requirements`.

---

## Step 4: Show Plan and Get Approval

Display `.build/PLAN.md` and ask:

> **Plan and e2e requirements look good?** Reply `yes` to start building, or tell me what to change.

Update and re-show if needed.

---

## Step 5: Create Team and Build

When approved, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Starting build...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### For each phase:

1. **Create a task** with `TaskCreate`
2. **Spawn builder teammate** in parallel:
   - `name`: "builder-phase-N"
   - `subagent_type: "teams:teams-builder"`
   - Prompt: full plan + phase spec + previous commits
3. **Spawn validator teammate** in parallel:
   - `name`: "validator-phase-N"
   - `subagent_type: "teams:teams-validator"`
   - Prompt: phase spec + e2e requirements + how to test
4. **Wait for both** to complete (they work simultaneously)
5. **Check validator verdict** — PASS or FAIL
6. **If FAIL** — retry builder once with validator feedback, re-validate
7. **Update task and plan** with results
8. **Print phase status**

### When all phases complete:

Print final summary (done/partial/failed counts).
