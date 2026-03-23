---
name: teams-plan
description: "Plan and build a feature. Orchestrator plans, spawns sequential Sonnet builder subagents per task (with Playwright/Maestro verification), then an Opus reviewer, then a builder to apply fixes."
user-invocable: true
---

# Teams: Plan + Build

You are the planner and orchestrator. Your job: discuss the feature, create a plan, execute it with sequential builder subagents, review the result with an Opus reviewer, and apply fixes.

---

## Step 1: Discuss + Plan

Ask: **"What do you want to build?"**

Discuss with the user. Identify the target platform: **web** or **mobile** (this determines whether the builder uses Playwright or Maestro for verification).

**Task sizing:** Each task should be a meaningful, self-contained unit of work — something a developer could complete in one focused session.
- **Too small:** "Add a button", "rename a variable" — merge into a larger task.
- **Too big:** "Build the entire backend", "implement all API routes" — split them.
- **Right size:** "Implement user authentication (signup, login, JWT middleware)", "Build the product listing page with filtering and pagination".

**Prepare the build directory:**

```bash
mkdir -p .build
```

If `.build/PLAN.md` already exists, ask the user:
> **A plan already exists from a previous build. Overwrite it, or use `/teams:run` to resume?**

Write `.build/PLAN.md`:

```markdown
# Plan: [Feature Name]

Generated: [date]
Platform: web | mobile
Status: draft

## Tasks
1. [ ] Task 1: [Description]
2. [ ] Task 2: [Description]
3. [ ] Task 3: [Description]

## Acceptance Criteria
- [Criterion 1]
- [Criterion 2]

## Verification
Tool: Playwright | Maestro
Scenarios:
- [Scenario 1: name — steps — expected result]
- [Scenario 2: name — steps — expected result]
```

---

## Step 2: Optional Plan Review

After writing the draft plan, ask the user:

> **"Would you like another AI agent to review this plan for completeness and edge cases? (Recommended: Yes)"**

If **yes**:
1. **Check for Multi-CLI MCP:** Look for `mcp__Multi-CLI__Ask-Codex` in your available tools (use `ToolSearch` if needed).
   - If available: read `.build/PLAN.md` and call `mcp__Multi-CLI__Ask-Codex` with the plan content and the prompt: *"Review this implementation plan. Identify missing tasks, edge cases, or architectural gaps. Be concise."*
   - If not available: use the `Agent` tool to spawn a general-purpose subagent with `model: opus` and prompt it to review `.build/PLAN.md` for completeness, edge cases, and architectural gaps.
2. Evaluate the feedback. Incorporate valid findings into `.build/PLAN.md`.
3. Briefly tell the user what changed.

If **no**: skip to Step 3.

---

## Step 3: Get Approval

Display `.build/PLAN.md` and ask:

> **"Plan looks good? Reply `yes` to start, or tell me what to change."**

---

## Step 4: Execute — Sequential Builder Subagents

When approved:

1. Update `.build/PLAN.md` status to `approved`.
2. Capture the base commit SHA before building starts:
   ```bash
   git rev-parse HEAD
   ```
   Save this as `BASE_SHA` — you will pass it to the reviewer later.
3. Print:
   ```
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
     TEAMS  Starting build...
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ```

For **each task in order**, use the `Agent` tool to spawn a builder subagent:

```
Agent(
  subagent_type: "teams:teams-builder",
  model: "sonnet",
  prompt: "You are implementing Task [N] of [M]: [task description].

    Platform: [web|mobile]

    Full plan:
    [paste .build/PLAN.md content]

    Your task: implement Task [N] only. Verify it works using [Playwright|Maestro], then commit.
    If [Playwright|Maestro] tools are not available, run tests/lint instead and note that E2E verification was skipped."
)
```

Wait for the subagent to complete before starting the next. After each task, update `.build/PLAN.md` (change `[ ]` to `[x]` on success, `[!]` on failure) and print the task board:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  [N of M tasks complete]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: Project Setup          [done]
  ►  Task 2: Auth System            [building...]
  ○  Task 3: API Routes             [pending]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Status symbols:
- `✓` — completed
- `►` — in progress
- `✗` — failed
- `○` — pending

If a builder subagent fails, log it as failed and continue with the next task.

---

## Step 5: Opus Review

After all tasks complete, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Reviewing implementation...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Spawn the reviewer using the `Agent` tool:

```
Agent(
  subagent_type: "teams:teams-reviewer",
  model: "opus",
  prompt: "Review the implementation for: [feature name].

    Base commit (before build started): [BASE_SHA]
    Use `git diff [BASE_SHA]..HEAD` to see all changes.

    Full plan:
    [paste .build/PLAN.md content]

    Write your review to .build/REVIEW.md.
    If mcp__Multi-CLI__Ask-Codex is available, use it for a second opinion."
)
```

---

## Step 6: Apply Fixes

After the reviewer completes, read `.build/REVIEW.md`.

If there are blocking findings:
1. Print a summary of the review findings.
2. Spawn a fix-pass builder:
   ```
   Agent(
     subagent_type: "teams:teams-builder",
     model: "sonnet",
     prompt: "You are applying review fixes (not implementing a new task).

       Review findings to fix:
       [paste blocking findings from .build/REVIEW.md]

       Platform: [web|mobile]

       Fix each blocking issue. Verify the fixes work using [Playwright|Maestro].
       If verification tools are not available, run tests/lint instead.
       Commit all fixes together with message: 'fix: address review findings'."
   )
   ```
3. Print final summary when done.

If no blocking findings, print final summary directly.

Final summary format:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Build complete!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: ...
  ✓  Task 2: ...
  ✓  Review: [passed | N fixes applied]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Then suggest:
> **Build done. Run `/teams:verify` to walk through manual E2E verification.**
