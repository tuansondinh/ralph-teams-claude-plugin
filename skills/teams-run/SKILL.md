---
name: teams-run
description: "Resume building a single Teams plan. Orchestrator spawns sequential or parallel builder subagents per incomplete task, then an Opus reviewer, then applies fixes."
user-invocable: true
---

# Teams: Run (Resume Build)

You are the orchestrator. Resume an existing build by running all incomplete tasks, then reviewing and applying fixes.

---

## Step 1: Find the Plan

Read `.ralph-teams/PLAN.md`. If not found:
> `.ralph-teams/PLAN.md` not found. Use `/teams-plan` to create a plan first.

Identify:
- Plan ID (the `Plan ID:` field — e.g. `#2`)
- All tasks, their status (`[x]` = done, `[!]` = failed, `[ ]` = incomplete), their complexity annotation (`complexity: simple` or `complexity: standard`), and any `parallel-group` annotations
- Platform (web or mobile)
- Verification scenarios

Print the current state:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  RALPH-TEAMS Plan #[N] — Resuming — [N of M tasks already done]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: Project Setup          [done]
  ✓  Task 2: Auth System            [done]
  ○  Task 3: API Routes             [pending]
  ○  Task 4: Frontend               [pending]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Step 1.5: Choose Execution Mode

Ask the user:

> **"Would you like to run remaining tasks in parallel or sequential mode?**
> - **Sequential** (default): tasks run one at a time in order — safer, easier to debug
> - **Parallel**: independent task groups run simultaneously — faster, but tasks must not share files or depend on each other
>
> The plan has [N] incomplete task(s), [M] of which are marked for parallel execution. Reply `parallel` or `sequential`."

Save the answer as `EXEC_MODE` (`parallel` or `sequential`).

---

## Step 2: Execute — Builder Subagents

Capture the base commit SHA before building starts:
```bash
git rev-parse HEAD
```
Save this as `BASE_SHA`.

**Group incomplete tasks into execution batches:**
- Tasks without `parallel-group` → each is its own batch (always sequential)
- Tasks sharing the same `parallel-group` label → form one batch
- Batches execute in the order the first task of each batch appears in the plan
- Skip already-completed tasks (`[x]`)

Pick the model from the task's complexity annotation:
- `complexity: simple` → `model: "haiku"`
- `complexity: standard` → `model: "sonnet"`

**For each batch, execute as follows:**

**If `EXEC_MODE = sequential` OR batch has only 1 task:**

Spawn one builder at a time and wait for completion:

```
Agent(
  subagent_type: "teams:teams-builder",
  model: "[haiku | sonnet based on task complexity]",
  prompt: "You are implementing Task [N] of [M]: [task description].

    Subtasks to complete:
    [list subtasks from the task]

    Platform: [web|mobile]

    Full plan:
    [paste .ralph-teams/PLAN.md content]

    Your task: implement Task [N] only, completing all its subtasks. Verify it works using [Playwright|Maestro], then commit.
    If [Playwright|Maestro] tools are not available, run tests/lint instead and note that E2E verification was skipped."
)
```

**If `EXEC_MODE = parallel` AND batch has 2+ tasks:**

Spawn all tasks in the batch simultaneously using `run_in_background: true`, then wait for all to complete before proceeding:

```
# Spawn all tasks in the batch at the same time:
Agent(
  subagent_type: "teams:teams-builder",
  model: "[haiku | sonnet]",
  run_in_background: true,
  name: "builder-task-[N]",
  prompt: "You are implementing Task [N] of [M] (running in parallel with Task [N2]): [task description].

    Subtasks to complete:
    [list subtasks]

    Platform: [web|mobile]

    Full plan:
    [paste PLAN.md content]

    IMPORTANT: You are running in parallel with other builders. Only modify files for your specific task.
    Do not modify shared config files, package.json, or files touched by parallel tasks.
    Your task: implement Task [N] only. Verify it works, then commit with message: 'feat: [task name]'."
)

# ... spawn all other tasks in the batch with run_in_background: true ...

# Then wait for all background agents in this batch to complete before moving to the next batch.
```

After each batch completes, update `.ralph-teams/PLAN.md` (change `[ ]` to `[x]` on success, `[!]` on failure for each task) and reprint the task board:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  RALPH-TEAMS  [N of M tasks complete]  [SEQUENTIAL | PARALLEL]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: Project Setup          [done]        (haiku)
  ►  Task 2: Auth System            [building...] (sonnet) ┐ parallel-group A
  ►  Task 3: DB Schema              [building...] (haiku)  ┘
  ○  Task 4: API Routes             [pending]     (sonnet)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If a builder subagent fails, log it as failed and continue with the next batch.

---

## Step 3: Opus Review

After all tasks complete, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  RALPH-TEAMS  Reviewing implementation...
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

    Plan file: .ralph-teams/PLAN.md
    Full plan:
    [paste .ralph-teams/PLAN.md content]

    Append your review to .ralph-teams/PLAN.md as a '## Review' section."
)
```

---

## Step 4: Apply Fixes

Read the `## Review` section from `.ralph-teams/PLAN.md`. If there are blocking findings:
1. Print a summary of the findings.
2. Spawn a fix-pass builder:
   ```
   Agent(
     subagent_type: "teams:teams-builder",
     model: "sonnet",
     prompt: "You are applying review fixes (not implementing a new task).

       Review findings to fix (from '## Review' section of .ralph-teams/PLAN.md):
       [paste blocking findings]

       Platform: [web|mobile]

       Fix each blocking issue. Verify the fixes work using [Playwright|Maestro].
       If verification tools are not available, run tests/lint instead.
       Commit all fixes together with message: 'fix: address review findings'."
   )
   ```
3. After the fix-pass builder completes, append a fix summary to `.ralph-teams/PLAN.md`:
   ```markdown
   ---

   ## Review Fixes Applied

   Fixes: [brief description of what was changed]
   Commit: fix: address review findings
   Status: All blocking findings resolved
   ```
   Also update the `## Review` section in the plan to mark any blocking findings as `Resolved` inline.

Final summary:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  RALPH-TEAMS  Plan #[N] — Build complete!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: ...
  ✓  Task 2: ...
  ✓  Review: [passed | N fixes applied]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Then suggest:
> **Build done. Run `/teams-verify` to walk through manual E2E verification.**

---

## Step 5: Optional — Update Docs

Ask the user:

> **"Would you like to update your documentation? Run `/teams-document` to have the scribe update your README, ARCHITECTURE.md, and other docs."**

If yes, invoke the `teams-document` skill.

If **no**, skip.
