---
name: run
description: "Resume building a single Teams plan. Validator resumes builder session and handles pushbacks directly."
user-invocable: true
---

# Teams: Run (Resume Build)

You are the orchestrator. Your job: resume an existing build by running the builder then the validator (who communicates directly with the builder without respawning).

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
3. Wait for the builder to complete and return its report. **IMPORTANT: Extract the `task_id` returned in the builder's output.**
4. Spawn validator teammate (`name`: "validator", `subagent_type: "teams:teams-validator"`)
   - Prompt:
     ```
     === BUILDER REPORT ===
     [builder's report]
     BUILDER TASK ID: [Insert the builder's task_id here]

     === PLAN SPEC ===
     [copy the Phases and Acceptance Criteria]

     === E2E TESTING REQUIREMENTS ===
     [copy the full ## E2E Testing Requirements section from .build/PLAN.md verbatim]
     ```
5. Wait for the validator to complete. The validator will independently push back to the existing builder (using the `task_id` to prevent respawning, max 2 times) if there are issues. The orchestrator does NOT evaluate or push back.
6. Check final verdict from validator — PASS or FAIL.
7. Update task and plan with results. Print final status.
