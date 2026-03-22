---
name: run
description: "Resume building a single Teams plan. Validator waits for builder to finish and handles pushbacks directly."
user-invocable: true
---

# Teams: Run (Resume Build)

You are the orchestrator. Your job: resume an existing build by running the builder then the validator (who handles pushbacks).

---

## Step 1: Find the Plan

```bash
cat .build/PLAN.md
```

If not found, tell user:
> `.build/PLAN.md` not found. Use `/teams:plan` to create a plan first.

---

## Step 2: Build

Create the team with `TeamCreate`:
- `team_name`: slugified from plan title
- `description`: "[Feature name] — agent team execution"

1. Create a task with `TaskCreate`
2. Spawn builder teammate (`name`: "builder", `subagent_type: "teams:teams-builder"`)
   - Prompt: full plan
3. Wait for the builder to complete and return its report (with commit SHA).
4. Spawn validator teammate (`name`: "validator", `subagent_type: "teams:teams-validator"`)
   - Prompt:
     ```
     === BUILDER REPORT ===
     [builder's report]

     === PLAN SPEC ===
     [copy the Goal, Tasks, and Acceptance Criteria]

     === E2E TESTING REQUIREMENTS ===
     [copy the full ## E2E Testing Requirements section from .build/PLAN.md verbatim]
     ```
5. Wait for the validator to complete. The validator will independently push back to the builder (max 2 times) if there are issues. The orchestrator does NOT evaluate or push back.
6. Check final verdict from validator — PASS or FAIL.
7. Update task and plan with results. Print final status.
