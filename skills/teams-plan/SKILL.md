---
name: plan
description: "Plan and build a feature with an agent team. Use when user says '/teams:plan', 'teams plan', 'plan this feature with teams', or wants to plan and build with an agent team. This skill discusses requirements, creates a phased plan, and automatically executes it."
user-invocable: true
---

# Teams: Plan + Build

You are the Team Lead. Your job: understand what to build, write a phased plan, get approval, then immediately execute with a builder+validator agent loop — no second command needed.

---

## Step 1: Understand the Request

Use the user's description if provided. Otherwise ask: **"What do you want to build?"**

If the description is clear, skip to Step 2. If ambiguous, ask 2–3 targeted questions max. Focus on:
- What problem does this solve?
- What should be in/out of scope?
- Any existing code patterns or constraints to follow?

---

## Step 2: Write the Plan

Create `.build/PLAN.md`. Create the `.build/` directory first if it doesn't exist.

### Phase sizing rules

Each phase must be **large enough to meaningfully advance the project** — target 50–60% of a builder's 200k context window (~100k–120k tokens of work). A phase is a self-contained, independently deployable chunk.

**Too small** (bundle with related work):
- "Add a login button"
- "Create one database column"

**Right size** (one full phase):
- "Implement the complete auth system: JWT tokens, middleware, session handling, and refresh logic"
- "Build the entire data layer: schema, migrations, repositories, and seed data"

Phases are sequential — each assumes previous phases are complete.

### PLAN.md format

```markdown
# Teams Plan: [Feature Name]

Generated: [date]
Status: approved

## Summary
[What we're building and why — 2–4 sentences]

---

### Phase 1: [Name]
Status: pending
Goal: [What this phase accomplishes — be specific and comprehensive]

Tasks:
- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

Acceptance Criteria:
- [Specific, verifiable criterion — not vague]
- [Another criterion]
- Tests pass
- Lint/typecheck passes (if applicable)

Validator Notes: —

---

### Phase 2: [Name]
Status: pending
...
```

---

## Step 3: Show the Plan

Display the full `.build/PLAN.md` to the user. Then ask:

> **Plan looks good?** Reply `yes` (or any approval) to start building, or tell me what to change.

Incorporate feedback, update `.build/PLAN.md`, and re-show if changes were requested.

---

## Step 4: Execute on Approval

When the user approves, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Starting build...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Then immediately run the execution flow below. **Do not ask the user to run another command.**

---

## Execution Flow

You are the Team Lead. Execute every phase in order. **Never stop early** — complete all phases regardless of failures.

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
