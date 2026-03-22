---
name: plan
description: "Plan and build a feature with an agent team. Use when user says '/teams:plan', 'teams plan', 'plan this feature with teams', or wants to plan and build with an agent team. This skill discusses requirements, creates a phased plan, and automatically executes it."
user-invocable: true
---

# Teams: Plan + Build

You are the Team Lead. Your job: understand what to build, write a phased plan, get user approval, then immediately spawn a team-lead agent to execute the build autonomously.

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

## Step 4: Create Team and Spawn Lead

When the user approves, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Starting build...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Then:

1. **Create a team** using `TeamCreate`:
   - `team_name`: derive from the plan title, e.g., `"teams-auth-system"` (slugified, lowercase, hyphens)
   - `description`: "[Feature name] — agent team execution"

2. **Spawn the Team Lead agent** using `Agent`:
   - `team_name`: the team name you just created
   - `name`: "team-lead"
   - `subagent_type: "teams:teams-lead"`
   - `mode: "bypassPermissions"`
   - Do NOT use `run_in_background` (let it run synchronously)
   - `prompt`:
   ```
   The plan is ready. Execute all phases. Be verbose about what you're doing.

   Current directory: [working directory]
   Plan file: .build/PLAN.md

   Your job:
   1. Read .build/PLAN.md
   2. For each phase in order:
      - Print: "Phase N: [name] — starting"
      - Create a task with TaskCreate
      - Spawn a builder agent to implement
      - Print the builder's work and commit SHA
      - Spawn a validator to verify
      - Print the validator's verdict
      - If FAIL: retry builder, show new commit, re-validate
      - Update task status
      - Update .build/PLAN.md
      - Print phase result
   3. After all phases, print final summary with counts

   Output everything — the user is watching.
   ```

3. **Capture and display team-lead output** — spawn the team-lead with Agent, capture its response, and display it to the user:
   ```
   [Agent call with the prompt above]
   [Team-lead response] → display this fully to the user
   ```
   The team-lead returns everything it did: phases executed, builder commits, validator verdicts, retries, final summary.

---

## Notes

- The Team Lead runs autonomously — no user action needed
- Team Lead will create tasks via `TaskList` so you can track progress
- Team Lead messages the team with phase completions
- All code, commits, and results are saved to the repo — view progress in `.build/PLAN.md` as it updates
