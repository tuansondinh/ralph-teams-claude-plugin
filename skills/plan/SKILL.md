---
name: plan
description: "Plan and build a feature with an agent team. Use when user says '/teams:plan', 'teams plan', 'plan this feature with teams', or wants to plan and build with an agent team."
user-invocable: true
---

# Teams: Plan + Build

You are a skill that creates plans and spawns teams. Your job:
1. Understand what to build
2. Write a phased plan in `.build/PLAN.md`
3. Get user approval
4. Create a team and spawn the team-lead teammate
5. Return (team-lead takes it from here)

---

## Step 1: Understand the Request

Ask: **"What do you want to build?"**

If ambiguous, ask 2–3 clarifying questions. Otherwise, proceed to Step 2.

---

## Step 2: Write the Plan

Create `.build/PLAN.md` with phases. Each phase should be ~100k–120k tokens of work (50–60% of builder's context).

Format:
```markdown
# Teams Plan: [Feature Name]

Generated: [date]
Status: approved

## Summary
[What we're building — 2–4 sentences]

---

### Phase 1: [Name]
Status: pending
Goal: [What this phase accomplishes]

Tasks:
- [ ] Task 1
- [ ] Task 2

Acceptance Criteria:
- Tests pass
- Lint/typecheck passes
- [Any specific requirements]

Validator Notes: —

---

### Phase 2: [Name]
Status: pending
...
```

---

## Step 3: Get Approval

Display `.build/PLAN.md` and ask:

> **Plan looks good?** Reply `yes` to start building, or tell me what to change.

Update and re-show if needed.

---

## Step 4: Create Team and Spawn Team Lead

When approved, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Starting build...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Then:

### 1. Create the team

Use `TeamCreate`:
- `team_name`: slugified from plan title, e.g., `"teams-auth-system"`
- `description`: "[Feature name] — agent team execution"

### 2. Spawn the team-lead teammate

Use `Agent` with:
- `team_name`: the team name from step 1
- `name`: "team-lead"
- `subagent_type: "teams:teams-lead"`
- `mode: "bypassPermissions"`

Prompt:
```
Read .build/PLAN.md and orchestrate the complete build.

Your job:
1. For each phase, spawn a builder teammate to implement it
2. After builder completes, spawn a validator teammate to verify
3. If validator fails, retry builder with feedback, then re-validate
4. Update .build/PLAN.md and tasks after each phase
5. When all phases are done, report completion

Go!
```

### 3. Display progress as the team works

The team-lead teammate now runs independently and messages progress. Display its messages and task updates to the user in real-time.

As progress comes in:
- Team-lead messages about phase starts, builder commits, validator verdicts, retries
- Display each message as it arrives
- Periodically read `.build/PLAN.md` and show phase status updates
- When team-lead reports completion, show final summary

Print progress like:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Phase 1: Build Structure — builder spawned
  Builder commit: abc123def456
  Validator spawned
  Verdict: PASS
  Phase 1 complete ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

When the team-lead is done, it will message completion and `.build/PLAN.md` will be fully updated.
