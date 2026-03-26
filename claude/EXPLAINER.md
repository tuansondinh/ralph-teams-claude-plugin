# ralph-teams Plugin — Complete Explanation

## What Is It?

**ralph-teams** is a Claude Code plugin that acts as an AI-powered engineering team. Instead of you writing code yourself, you describe a feature and the plugin orchestrates multiple specialized AI agents to plan, build, review, verify, and document it — all tracked in a single file.

It installs in Claude Code with two commands:
```
/plugin marketplace add tuansondinh/ralph-teams-claude-plugin
/plugin install ralph-teams@ralph-teams-claude-plugin
```

---

## The Core Concept: Agent Orchestration

The plugin is built entirely in **Markdown files with YAML frontmatter** — no compiled code, no build step. Claude Code reads these files as instructions for how to behave. There are two types of components:

- **Skills** (`skills/`) — commands the user invokes, like `/teams:plan`
- **Agents** (`agents/`) — subagents spawned by skills to do specialized work

---

## The Full Lifecycle

### 1. `/teams:plan` — Plan + Build

This is the entry point. The orchestrator (main Claude) does the following:

**a) Discuss the feature**
- Asks: "What do you want to build?"
- Identifies the target platform (web or mobile)
- Determines the right phase breakdown

**b) Write a Plan File** (`.ralph-teams/PLAN-N.md`)
The plan is a single Markdown document that tracks the entire feature lifecycle. It contains:
- **Phases** — major areas of work (e.g., "Auth System", "API Routes")
- **Tasks** — concrete steps within each phase
- **Complexity** — `simple` (uses Haiku model) or `standard` (uses Sonnet)
- **Parallel groups** — phases that can safely run simultaneously
- **Acceptance Criteria** — what "done" means
- **Verification Scenarios** — E2E test scripts for manual or automated testing

**c) Optional AI Plan Review**
The plan is optionally sent to an Opus agent (or Codex CLI) for a second opinion on completeness and edge cases before you approve.

**d) User Approval**
You review the plan and say "yes" to proceed. You also choose: **sequential** (one phase at a time, safer) or **parallel** (independent phases run simultaneously, faster).

**e) Build Execution — Builder Subagents**
For each phase, the orchestrator spawns a fresh **Haiku or Sonnet builder agent** with a clean 200k token context window. Each builder:
1. Reads the plan for context
2. **Writes tests first** (TDD — tests must fail before implementation)
3. Implements the phase tasks in order
4. **Verifies** using Playwright (web) or Maestro (mobile)
5. Commits with a descriptive message
6. Reports back to the orchestrator

In parallel mode, independent phase groups are spawned simultaneously with `run_in_background: true`, then the orchestrator waits for all to complete before proceeding.

**f) Opus Review**
After all phases are built, an **Opus reviewer agent** is spawned. It:
- Runs `git diff BASE_SHA..HEAD` to see all changes
- Checks every acceptance criterion
- Runs the test suite
- Optionally asks Codex CLI for a second opinion on complex changes
- Appends a `## Review` section to the plan file with: `PASS` or `NEEDS FIXES`, blocking vs. non-blocking findings, and test status

**g) Apply Fixes**
If there are blocking findings, a final Sonnet builder is spawned to fix them and commit.

---

### 2. `/teams:run` — Resume

If a build was interrupted, `/teams:run` reads the plan file, finds incomplete phases (marked `[ ]`), and resumes from where it left off. Same execution flow as above.

---

### 3. `/teams:verify` — Manual E2E Verification

After the automated build, you verify it works with your own eyes. The skill:
- Loads the plan and its verification scenarios
- Walks you through each scenario one at a time, showing the exact steps
- Records your `pass / fail / skip` response
- If something fails, immediately offers to invoke `/teams:debug`
- Appends a `## Verification` section to the plan file

---

### 4. `/teams:debug` — Bug Fixing

Can be triggered anytime — by the user or automatically from `/teams:verify` on failure. It:
- Loads the full plan (which includes all prior review and verification history)
- Confirms the bug description with you
- Spawns a targeted Sonnet builder to investigate root cause and fix it
- Verifies the fix with Playwright/Maestro
- Commits and appends a `## Debug Fix` entry to the plan

---

### 5. `/teams:document` — Update Docs

Spawns a Haiku scribe agent to update existing project documentation (README, ARCHITECTURE.md, etc.) to reflect what was built. Appends a `## Documentation` section to the plan.

---

## The Plan File as the Single Source of Truth

Every stage appends to one file per feature — `.ralph-teams/PLAN-N.md`. Nothing is overwritten, only appended. By the end of a feature, the file contains:

| Section | Written by |
|---------|-----------|
| Phases, criteria, scenarios | Orchestrator (plan creation) |
| `## Review` | Opus reviewer |
| `## Review Fixes Applied` | Orchestrator |
| `## Verification` | Verify skill |
| `## Debug Fix` | Debug skill (each bug) |
| `## Documentation` | Document skill |

This gives you a full audit trail of the entire feature from plan to production.

---

## Model Selection Strategy

The plugin intelligently picks the right model for each job:

| Agent | Model | When |
|-------|-------|------|
| Builder | Haiku | Phase complexity = `simple` (config tweaks, renames) |
| Builder | Sonnet | Phase complexity = `standard` (all real logic) |
| Reviewer | Opus | Always — it's the QA pass |
| Fix builder | Sonnet | Always |
| Scribe (docs) | Haiku | Documentation updates |

---

## Architecture Summary

```
ralph-teams-plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── agents/
│   ├── teams-builder.md     # Sonnet/Haiku builder — implements phases
│   └── teams-reviewer.md    # Opus reviewer — QA pass
└── skills/
    ├── teams-plan/SKILL.md  # Main orchestrator — discuss → plan → build → review
    ├── teams-run/SKILL.md   # Resume an existing plan
    ├── teams-verify/SKILL.md # Manual E2E walkthrough
    ├── teams-document/SKILL.md # Doc updater
    └── debug/SKILL.md       # Bug fixer
```

**Everything is Markdown.** No TypeScript, no compilation, no package manager. Claude Code reads the Markdown instructions and executes them as agent behavior. The plugin is essentially a programming language written in plain English.

---

## Key Design Principles

1. **One fresh agent per phase** — each builder starts with a clean 200k context, no context pollution between phases
2. **TDD by default** — builders write failing tests before implementing, ensuring correctness
3. **Verification is mandatory** — Playwright/Maestro runs before every commit
4. **Single lifecycle document** — the plan file is the complete record, no scattered files
5. **Resumable** — commit after each phase means you can always pick up where you left off
6. **Model efficiency** — Haiku for simple work, Sonnet for standard work, Opus only for review
