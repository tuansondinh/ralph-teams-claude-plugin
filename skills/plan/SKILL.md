---
name: plan
description: "Plan and build a feature with an agent team. Builder runs first, then validator reviews and pushes back if needed."
user-invocable: true
---

# Teams: Plan + Build

You are the orchestrator. Your job:
1. Understand what to build
2. Write a phased plan
3. Collect e2e testing requirements from user
4. Get approval
5. Create team and orchestrate builder then validator for each phase

---

## Step 1: Understand the Request

Ask: **"What do you want to build?"**

If unclear, ask 2–3 clarifying questions. Otherwise proceed.

---

## Step 2: Write the Plan

Create `.build/PLAN.md` with phases (~100k–120k tokens per phase).

**Dependency analysis:** Before writing phases, identify which phases can run in parallel. Phases that don't depend on each other's output get the same `Group` number. Phases that must complete before others proceed get a higher group number. Assign group numbers starting at 1.

```markdown
# Teams Plan: [Feature Name]

Generated: [date]
Status: approved
Mode: sequential

## Summary
[What we're building — 2–4 sentences]

## Execution Order

| Group | Phases | Notes |
|-------|--------|-------|
| 1     | Phase 1: [Name] | — |
| 2     | Phase 2: [Name], Phase 3: [Name] | ⚡ run in parallel |
| 3     | Phase 4: [Name] | depends on group 2 |

## E2E Testing Requirements
[To be collected from user]

---

### Phase 1: [Name]
Group: 1
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
Group: 2
Status: pending
Goal: ...

---

### Phase 3: [Name]
Group: 2
Status: pending
Goal: ...
```

> **Rule:** Only assign the same group to phases that are truly independent — they must not read or write the same files/state. When in doubt, use sequential groups.

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

> **Plan and e2e requirements look good?**
>
> Also — should independent phases run in **parallel** or **sequential**?
> - `parallel` — phases in the same group run simultaneously (faster, uses more resources)
> - `sequential` — one phase at a time in order (safer, easier to debug)
>
> Reply with your choice + `yes` to start, or tell me what to change.

Set `Mode: parallel` or `Mode: sequential` in `.build/PLAN.md` based on the user's answer (default: `sequential`). Update and re-show if needed.

---

## Step 5: Create Team and Build

When approved, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Starting build...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Read `Mode:` from `.build/PLAN.md` and execute accordingly:

---

#### Mode: sequential — for each phase in order:

1. Create a task with `TaskCreate`
2. Spawn builder teammate (`name`: "builder-phase-N", `subagent_type: "teams:teams-builder"`)
   - Prompt: full plan + phase spec + previous phase summaries
3. Wait for builder to complete and get its report (commit SHA).
4. Spawn validator teammate (`name`: "validator-phase-N", `subagent_type: "teams:teams-validator"`)
   - Prompt:
     ```
     === BUILDER REPORT ===
     [Builder's commit SHA and summary]

     === PHASE SPEC ===
     [current phase: name, goal, tasks, acceptance criteria]

     === E2E TESTING REQUIREMENTS ===
     [copy the full ## E2E Testing Requirements section from .build/PLAN.md verbatim]
     ```
5. Wait for the validator to complete. The validator will push back to the builder directly if needed.
6. Check validator verdict — PASS or FAIL. Update task and plan with results. Print phase status. Move to next phase.

---

#### Mode: parallel — for each group (in ascending group number order):

1. Collect all phases in this group with `Status: pending` or `Status: partial`
2. For each phase in the group, **simultaneously**:
   - Create a task with `TaskCreate`
   - Spawn builder teammate (`name`: "builder-phase-N", `subagent_type: "teams:teams-builder"`)
     - Prompt: full plan + phase spec + previous phase summaries
   - Wait for the builder to complete and get its report (commit SHA).
   - Spawn validator teammate (`name`: "validator-phase-N", `subagent_type: "teams:teams-validator"`)
     - Prompt:
       ```
       === BUILDER REPORT ===
       [Builder's commit SHA and summary]

       === PHASE SPEC ===
       [current phase: name, goal, tasks, acceptance criteria]

       === E2E TESTING REQUIREMENTS ===
       [copy the full ## E2E Testing Requirements section from .build/PLAN.md verbatim]
       ```
   - Wait for the validator to complete. (The validator pushes back to the builder directly if needed).
3. Wait for **all phases in the group** to complete before starting the next group
4. For each completed phase: check validator verdict (PASS or FAIL), update task and plan. The orchestrator does not push back.
5. Print group status. Move to next group.

---

### When all phases complete:

Print final summary (done/partial/failed counts).
