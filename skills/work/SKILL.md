---
name: work
description: "Resume building an existing Teams plan. Validator waits for builder to finish and handles pushbacks directly."
user-invocable: true
---

# Teams: Work (Resume Build)

You are the orchestrator. Your job: resume an existing build. For each incomplete phase, run the builder then the validator (who handles pushbacks).

---

## Step 1: Find the Plan

```bash
cat .build/PLAN.md
```

If not found, tell user:
> `.build/PLAN.md` not found. Use `/teams:plan` to create a plan first.

Extract:
- Feature name
- Phase count
- Which phases are done, pending, or partial?
- `Mode:` field (`sequential` or `parallel`, default: `sequential`)
- The `Group:` number for each phase (only relevant for `parallel` mode)

---

## Step 2: Build Incomplete Phases

Create the team with `TeamCreate`:
- `team_name`: slugified from plan title
- `description`: "[Feature name] — agent team execution"

Read `Mode:` from `.build/PLAN.md` and execute accordingly:

---

#### Mode: sequential — for each incomplete phase in order:

1. Create a task with `TaskCreate`
2. Spawn builder teammate (`name`: "builder-phase-N", `subagent_type: "teams:teams-builder"`)
   - Prompt: full plan + phase spec + previous phase summaries
3. Wait for the builder to complete and return its report (with commit SHA).
4. Spawn validator teammate (`name`: "validator-phase-N", `subagent_type: "teams:teams-validator"`)
   - Prompt:
     ```
     === BUILDER REPORT ===
     [builder's report]

     === PHASE SPEC ===
     [current phase: name, goal, tasks, acceptance criteria]

     === E2E TESTING REQUIREMENTS ===
     [copy the full ## E2E Testing Requirements section from .build/PLAN.md verbatim]
     ```
5. Wait for the validator to complete. The validator will independently push back to the builder (max 2 times) if there are issues. The orchestrator does NOT evaluate or push back.
6. Check final verdict from validator — PASS or FAIL.
7. Update task and plan with results. Print phase status. Move to next phase.

---

#### Mode: parallel — for each group (in ascending group number order):

1. Collect all incomplete phases in this group
2. For each phase in the group, **simultaneously**:
   - Create a task with `TaskCreate`
   - Spawn builder teammate (`name`: "builder-phase-N", `subagent_type: "teams:teams-builder"`)
     - Prompt: full plan + phase spec + previous phase summaries
3. Wait for **all builders in the group** to complete.
4. For each completed builder, **simultaneously**:
   - Spawn validator teammate (`name`: "validator-phase-N", `subagent_type: "teams:teams-validator"`)
     - Prompt:
       ```
       === BUILDER REPORT ===
       [builder's report]

       === PHASE SPEC ===
       [current phase: name, goal, tasks, acceptance criteria]

       === E2E TESTING REQUIREMENTS ===
       [copy the full ## E2E Testing Requirements section from .build/PLAN.md verbatim]
       ```
5. Wait for **all validators in the group** to complete before the next group. (Validators will handle any pushbacks to their builders directly, up to 2 times).
6. For each completed phase: check final verdict (PASS or FAIL), update task and plan. The orchestrator does NOT evaluate or push back.
7. Print group status. Move to next group.

---

## When all phases complete:

Print final summary with counts and move on.
