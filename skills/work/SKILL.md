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

---

## Step 2: Build Incomplete Phases

Create the team with `TeamCreate`:
- `team_name`: slugified from plan title
- `description`: "[Feature name] — agent team execution"

### For each incomplete phase (status: pending or partial):

1. **Create a task** with `TaskCreate`

2. **Spawn builder teammate** in parallel:
   - `name`: "builder-phase-N"
   - `subagent_type: "teams:teams-builder"`
   - Prompt: full plan + phase spec + previous commits

3. **Spawn validator teammate** in parallel:
   - `name`: "validator-phase-N"`
   - `subagent_type: "teams:teams-validator"`
   - Prompt: phase spec + e2e requirements + how to test

4. **Wait for both** to complete (they work simultaneously)

5. **Check validator verdict** — PASS or FAIL

6. **If FAIL** — retry builder once with validator feedback, re-validate

7. **Update task and plan** with results

8. **Print phase status**

---

## When all phases complete:

Print final summary with counts and move on.
