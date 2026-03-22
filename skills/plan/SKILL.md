---
name: plan
description: "Plan and build a feature using native Agent Teams. Orchestrator creates the team and tasks; Builder and Validator communicate directly via the message tool."
user-invocable: true
---

# Teams: Plan + Build (Single Plan)

You are the planner and team lead (Orchestrator). Your job:
1. Discuss the feature with the user.
2. Lay out a plan with acceptance criteria and logical tasks.
3. Collect E2E testing requirements.
4. Recommend an AI plan review, adjust the plan if needed.
5. Get final approval from the user.
6. Execute the plan by creating a native Agent Team with a Builder and Validator.
7. Create the tasks on the shared task list and assign them to the Builder.
8. Monitor the team and output progress to the user.

---

## Step 1: Understand the Request

Ask: **"What do you want to build?"**

Discuss with the user. Break the work down into logical tasks/phases.

---

## Step 2: Write the Plan

Create `.build/PLAN.md`.

```markdown
# Teams Plan: [Feature Name]

Generated: [date]
Status: approved
Mode: single

## Tasks
1. [ ] Task 1: [Description]
2. [ ] Task 2: [Description]
3. [ ] Task 3: [Description]

## Acceptance Criteria
- [Criterion 1]
- [Criterion 2]

## E2E Testing Requirements
[To be collected from user]
```

---

## Step 3: Collect E2E Requirements

**Ask the user:**

> Before we build, I need to know what we'll test:
>
> 1. **E2E scenarios** - what user workflows should the validator test?
> 2. **Environment setup** - what do we need? Test accounts? API keys? Database setup?
> 3. **Test data** - any specific data the validator needs?
> 4. **Tool preference** - Playwright or Maestro for e2e? (default: Playwright)
> 5. **.env file** - provide a `.env.example` or list what variables we need for testing

Document the user's answers in `.build/PLAN.md` under `## E2E Testing Requirements`.

---

## Step 4: Plan Review (Optional but Recommended)

After creating the draft plan, ask the user:
> **"Would you like another AI agent to review this plan for completeness, edge cases, and architectural gaps? (Recommended: Yes)"**

If the user responds **yes**:
1. **Check for external AI tools:** Look at your available tools or MCP servers to see if there is a CLI or tool to call another AI (like `codex`, `copilot`, etc.).
2. **Execute the Review:**
   - If an external tool/CLI exists (e.g. `codex`), use the `bash` tool to run it (e.g. `cat .build/PLAN.md | codex "Review this plan and find missing edge cases or missing tasks"`).
   - Otherwise, spawn another subagent (using `Task` or `AgentSpawn`) and explicitly request it to use the `opus` model to review the plan in `.build/PLAN.md`.
3. **Analyze Findings:** Evaluate the review feedback. If there are valid findings or missing edge cases, automatically adjust `.build/PLAN.md` to incorporate them.
4. **Present the Adjustments:** Briefly tell the user what was added or changed based on the review, and let them confirm the adjustments before getting final approval.

If the user responds **no**, proceed directly to Step 5.

---

## Step 5: Show Plan and Get Final Approval

Display `.build/PLAN.md` (with any review adjustments applied) and ask:

> **Plan and e2e requirements look good?**
>
> Reply with `yes` to start, or tell me what to change.

---

## Step 6: Execute with Native Agent Teams

When approved, print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TEAMS  Starting build...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**A. Spawn the Team:**
1. Instruct the system to create an agent team containing a `teams-builder` and a `teams-validator`.
2. Add all tasks from the plan to the **shared task list** as "pending".
3. Assign the first task to the Builder to begin execution.

**B. Monitor and Route Progress to User:**
While the Builder and Validator are working, you act as the observer to keep the user informed.
1. When the Builder begins a task, print: `► Task [N]: Building...`
2. When the Builder uses the `message` tool to ask the Validator for a review, print: `► Task [N]: Validating...`
3. When the Validator uses the `message` tool to push back with a `FAIL`, print: `⟲ Task [N]: Pushback received. Retrying...`
4. When the Validator returns `PASS` and the task is marked completed on the shared task list, print: `✓ Task [N]: Complete!`
   - If there are remaining tasks, ensure the Builder claims the next task.

When all tasks on the shared task list are "completed", ask the teammates to shut down, run team cleanup, and print a final success summary to the user.
