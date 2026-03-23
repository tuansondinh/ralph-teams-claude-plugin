---
name: teams-run
description: "Resume building a single Teams plan. Orchestrator spawns sequential Sonnet builder subagents per incomplete task, then an Opus reviewer, then applies fixes."
user-invocable: true
---

# Teams: Run (Resume Build)

You are the orchestrator. Resume an existing build by running all incomplete tasks, then reviewing and applying fixes.

---

## Step 1: Find the Plan

Read `.build/PLAN.md`. If not found:
> `.build/PLAN.md` not found. Use `/teams:plan` to create a plan first.

Identify:
- All tasks and their status (`[x]` = done, `[!]` = failed, `[ ]` = incomplete)
- Platform (web or mobile)
- Verification scenarios

Print the current state:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Resuming — [N of M tasks already done]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: Project Setup          [done]
  ✓  Task 2: Auth System            [done]
  ○  Task 3: API Routes             [pending]
  ○  Task 4: Frontend               [pending]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Step 2: Execute — Sequential Builder Subagents

Capture the base commit SHA before building starts:
```bash
git rev-parse HEAD
```
Save this as `BASE_SHA`.

For **each incomplete task** (`[ ]` or `[!]`) **in order**, use the `Agent` tool:

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

Wait for each subagent to complete before starting the next. After each task, update `.build/PLAN.md` (change `[ ]` to `[x]` on success, `[!]` on failure) and reprint the task board.

If a builder subagent fails, log it as failed and continue.

---

## Step 3: Opus Review

After all tasks complete, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Reviewing implementation...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Spawn the reviewer:

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

## Step 4: Apply Fixes

Read `.build/REVIEW.md`. If there are blocking findings:
1. Print a summary of the findings.
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

Final summary:
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
