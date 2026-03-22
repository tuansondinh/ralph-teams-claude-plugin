---
name: teams-work
description: "Execute an existing Teams build plan. Use when user says '/teams:work', 'run the teams build', 'execute the plan', 'resume the build', or wants to re-run or continue an existing .build/PLAN.md."
user-invocable: true
---

# Teams: Work

Execute the build plan in `.build/PLAN.md`. Use this to run or resume a plan that was already created by `/teams:plan`.

---

## Before Starting

1. Check `.build/PLAN.md` exists. If not:
   > "No plan found. Run `/teams:plan` first to create one."

2. Read the full plan.

3. Identify which phases need work:
   - Run phases with `Status: pending` or `Status: failed`
   - Skip phases with `Status: done` (unless user explicitly asked to redo)
   - Resume `Status: in-progress` phases from the beginning (treat as pending)

4. Print the current plan state and tell the user how many phases will run.

---

## Execution Flow

You are the Team Lead. Execute every pending/failed phase in order. **Never stop early** — complete all phases regardless of failures.

### For each phase:

**1. Print the progress display**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Phase N/Total — [Phase Name]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Phase 1: [Name]          [done]
  ►  Phase 2: [Name]          [building...]
  ○  Phase 3: [Name]          [pending]
  ○  Phase 4: [Name]          [pending]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Icons: `✓` done · `✗` failed · `~` partial · `►` in-progress · `○` pending

**2. Update PLAN.md**

Set the current phase `Status: in-progress`.

**3. Spawn the Builder**

Use the `Agent` tool with `subagent_type: "teams-builder"` and `mode: "bypassPermissions"`.

Pass this prompt:
```
=== FULL PLAN ===
[paste full contents of .build/PLAN.md]

=== YOUR PHASE ===
[paste the current phase section: goal, tasks, acceptance criteria]

=== PREVIOUS PHASES COMPLETED ===
[list commit SHAs and brief summaries of what prior phases built]
```

**4. Spawn the Validator**

Use the `Agent` tool with `subagent_type: "teams-validator"`.

Pass this prompt:
```
=== PHASE SPEC ===
[paste the current phase section: goal, tasks, acceptance criteria]

=== BUILDER'S COMMIT SHA ===
[commit SHA from builder's BUILDER REPORT]
```

**5. If FAIL — one retry**

If the validator returns `VERDICT: FAIL`:
- Spawn a new Builder (`subagent_type: "teams-builder"`) with the validator's findings attached as additional context
- Add to the builder prompt: `=== VALIDATOR FEEDBACK (apply these fixes) ===\n[findings]`
- Spawn a new Validator to re-check the new commit
- This is the **final attempt** — no further retries

**6. Update PLAN.md with result**

Based on the final validator verdict:
- PASS → `Status: done`
- FAIL after retry → `Status: partial` and write the validator findings into `Validator Notes:`

**7. Continue to next phase — never stop**

---

## Final Summary

After all phases are complete, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Build Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Phase 1: [Name]          [done]
  ✓  Phase 2: [Name]          [done]
  ~  Phase 3: [Name]          [partial]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Done: N   Partial: N   Failed: N
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If any phases are partial/failed, list the validator notes so the user knows what needs attention.
