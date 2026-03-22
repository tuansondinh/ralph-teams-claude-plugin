---
name: work
description: "Execute an existing Teams build plan. Use when user says '/teams:work', 'run the teams build', 'execute the plan', 'resume the build', or wants to re-run or continue an existing .build/PLAN.md."
user-invocable: true
---

# Teams: Work (Resume Build)

You are the Team Lead. Your job: find an existing `.build/PLAN.md`, resume the build, and orchestrate builder and validator teammates for incomplete phases.

---

## Step 1: Find the Plan

```bash
cat .build/PLAN.md
```

If the file doesn't exist, tell the user:
> `.build/PLAN.md` not found. Use `/teams:plan` to create a plan first.

If it exists, extract:
- The feature name from the title
- Total phase count
- Current phases: which are done, pending, partial, failed?

---

## Step 2: Create Team and Resume

Print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Resuming build...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Then orchestrate the build for incomplete phases:

### 1. Create the team

Use `TeamCreate`:
- `team_name`: derive from the plan title the same way as in `/teams:plan`, e.g., `"teams-auth-system"` (slugified, lowercase, hyphens)
- `description`: "[Feature name] — agent team execution"

### 2. For each phase

**Skip if done**: if phase `Status: done`, print and continue to next phase.

**Resume if pending or partial**: if `Status: pending` or `Status: partial`, orchestrate:

Print the phase header:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Phase N: [Phase Name] — resuming
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**a) Create a task** using `TaskCreate`:
```
{
  "subject": "Build Phase N: [Phase Name]",
  "description": "[phase goal and acceptance criteria]"
}
```

**b) Spawn the builder** using `Agent`:
- `team_name`: the team name
- `name`: `"builder-phase-N"`
- `subagent_type: "teams:teams-builder"`
- `mode: "bypassPermissions"`
- `prompt`:
```
=== FULL PLAN ===
[entire .build/PLAN.md contents]

=== YOUR PHASE ===
[current phase: goal, tasks, acceptance criteria]

=== PREVIOUS PHASES COMPLETED ===
[list of prior phase commit SHAs and summaries, or "none"]
```

Display builder output to the user.

**c) Spawn the validator** using `Agent`:
- `team_name`: the team name
- `name`: `"validator-phase-N"`
- `subagent_type: "teams:teams-validator"`
- `prompt`:
```
=== PHASE SPEC ===
[current phase: goal, tasks, acceptance criteria]

=== BUILDER'S COMMIT SHA ===
[commit SHA from builder report]
```

**d) Check the verdict** — extract `VERDICT: PASS` or `VERDICT: FAIL`.

**e) If FAIL — retry once**:
- Spawn a new builder with validator findings
- Get new commit SHA
- Re-validate
- Final attempt

**f) Update task and plan**:
- `TaskUpdate` to mark task `status: "completed"`
- Update `.build/PLAN.md`:
  - Set phase `Status: done` (if PASS) or `Status: partial` (if FAIL after retry)
  - Add validator notes if FAIL

Print phase result.

### 3. After all phases

Print final summary with completion counts.
