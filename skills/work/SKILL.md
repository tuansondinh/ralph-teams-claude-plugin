---
name: work
description: "Resume building an existing Teams plan. Builder and validator work in parallel."
user-invocable: true
---

# Teams: Work (Resume Build)

You are the orchestrator. Your job: resume an existing build where builder and validator work in parallel for each incomplete phase.

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
3. Spawn validator teammate alongside the builder (`name`: "validator-phase-N", `subagent_type: "teams:teams-validator"`)
   - Prompt:
     ```
     === PHASE SPEC ===
     [current phase: name, goal, tasks, acceptance criteria]

     === E2E TESTING REQUIREMENTS ===
     [copy the full ## E2E Testing Requirements section from .build/PLAN.md verbatim]
     ```
4. Wait for both to complete
5. Check verdict — PASS or FAIL. If FAIL — retry builder once with feedback, re-validate
6. Update task and plan with results. Print phase status. Move to next phase.

---

#### Mode: parallel — for each group (in ascending group number order):

1. Collect all incomplete phases in this group
2. For each phase in the group, **simultaneously**:
   - Create a task with `TaskCreate`
   - Spawn builder teammate (`name`: "builder-phase-N", `subagent_type: "teams:teams-builder"`)
     - Prompt: full plan + phase spec + previous phase summaries
   - Spawn validator teammate alongside the builder (`name`: "validator-phase-N", `subagent_type: "teams:teams-validator"`)
     - Prompt:
       ```
       === PHASE SPEC ===
       [current phase: name, goal, tasks, acceptance criteria]

       === E2E TESTING REQUIREMENTS ===
       [copy the full ## E2E Testing Requirements section from .build/PLAN.md verbatim]
       ```
3. Wait for **all phases in the group** to complete before the next group
4. For each completed phase: check verdict, retry if FAIL, update task and plan
5. Print group status. Move to next group.

---

## When all phases complete:

Print final summary with counts and move on.
