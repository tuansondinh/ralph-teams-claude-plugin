# ralph-teams-plugin — Codebase Reference

**Version:** 1.4.2 | **Type:** Claude Code Plugin (Markdown-based, no compiled code)

---

## File Map

```
ralph-teams-plugin/
├── .claude-plugin/
│   ├── plugin.json               # Plugin metadata (name, version, description)
│   └── marketplace.json          # Plugin registry metadata (must match plugin.json version)
├── agents/
│   ├── teams-builder.md          # Sonnet builder subagent (implements tasks + applies review fixes)
│   └── teams-reviewer.md         # Opus reviewer subagent (appends ## Review to plan file)
├── skills/
│   ├── teams-plan/SKILL.md       # /teams:plan — discuss → plan → build → review → fix
│   ├── teams-run/SKILL.md        # /teams:run  — resume an existing plan
│   ├── teams-verify/SKILL.md     # /teams:verify — manual E2E verification walkthrough
│   ├── teams-document/SKILL.md   # /teams:document — update project docs
│   └── debug/SKILL.md            # /teams:debug — fix bugs against active plan
├── README.md
├── FLOW.md                       # Mermaid execution flow diagram
└── codebase.md                   # This file
```

---

## Plugin Architecture

All files are **Markdown with YAML frontmatter** — no build step, no package manager.

### Agent Files (`agents/`)

Frontmatter fields: `name`, `description`, `model` (sonnet/opus/haiku).

| Agent | Model | Role |
|-------|-------|------|
| `teams-builder.md` | Sonnet | Implements a single task or applies review fixes. Verifies with Playwright (web) or Maestro (mobile) before committing. |
| `teams-reviewer.md` | Opus | Reviews the full implementation against acceptance criteria. Runs tests. Optionally consults Codex. Appends `## Review` section to the plan file. |

### Skill Files (`skills/`)

Frontmatter fields: `name`, `description`, `user-invocable: true`.

| Skill file | Command | Purpose |
|------------|---------|---------|
| `teams-plan/SKILL.md` | `/teams:plan` | Discuss → plan (with parallel-group annotations) → optional AI review → approve → choose sequential/parallel → build → Opus review → apply fixes |
| `teams-run/SKILL.md` | `/teams:run` | Resume a plan; choose sequential/parallel → run incomplete tasks → review + fix |
| `teams-verify/SKILL.md` | `/teams:verify` | Walk user through manual E2E verification; appends `## Verification` to plan |
| `teams-document/SKILL.md` | `/teams:document` | Spawn Haiku scribe to update project docs; appends `## Documentation` to plan |
| `debug/SKILL.md` | `/teams:debug` | Fix a bug against the active plan; appends `## Debug Fix` to plan |

---

## Execution Model

**Sequential mode:**
```
Orchestrator (teams-plan or teams-run)
  ├── Agent(teams-builder, haiku/sonnet) → Task 1 → verify → commit
  ├── Agent(teams-builder, haiku/sonnet) → Task 2 → verify → commit
  ├── ...
  ├── Agent(teams-reviewer, opus)        → appends ## Review to PLAN-N.md
  └── Agent(teams-builder, sonnet)       → apply review fixes → commit
```

**Parallel mode (tasks in same parallel-group run simultaneously):**
```
Orchestrator
  ├── Agent(teams-builder) → Task 1 → verify → commit        [sequential batch]
  ├── Agent(teams-builder, bg) → Task 2 ┐                    [parallel batch A]
  ├── Agent(teams-builder, bg) → Task 3 ┘ → both complete
  ├── Agent(teams-builder) → Task 4 → verify → commit        [sequential batch]
  ├── Agent(teams-reviewer, opus) → appends ## Review
  └── Agent(teams-builder, sonnet) → apply fixes → commit
```

- Model per task: `complexity: simple` → haiku, `complexity: standard` → sonnet
- Each builder verifies with **Playwright** (web) or **Maestro** (mobile) before committing
- If verification tools are unavailable, builders fall back to tests/lint
- The Opus reviewer runs after all tasks, reviewing the full diff from `BASE_SHA`
- The reviewer optionally uses Codex CLI for a second opinion (complex tasks only)
- If blocking findings exist, a final builder is spawned to apply fixes

---

## Plan File — Single Lifecycle Document

Each feature has one plan file: `.ralph-teams/PLAN-N.md`. Every stage appends a section to it — no separate `REVIEW.md` or `VERIFY.md`.

| Section | Written by | When |
|---------|-----------|------|
| Tasks, criteria, scenarios | Orchestrator | Plan creation |
| `## Review` | Reviewer agent | After build completes |
| `## Review Fixes Applied` | Orchestrator | After fix-pass builder |
| `## Verification` | Verify skill | After manual walkthrough |
| `## Debug Fix` | Debug skill | After each bug fix |
| `## Documentation` | Document skill | After scribe completes |

### PLAN-N.md format

```markdown
# Plan #N: [Feature Name]

Plan ID: #N
Generated: [date]
Platform: web | mobile
Status: draft | approved

## Phases
1. [ ] Phase 1: [Description] — complexity: simple
   - [Task 1]
   - [Task 2]
2. [ ] Phase 2: [Description] — complexity: standard — parallel-group: A
   - [Task 1]
3. [ ] Phase 3: [Description] — complexity: standard — parallel-group: A
   - [Task 1]

## Acceptance Criteria
- [Criterion 1]

## Verification
Tool: Playwright | Maestro
Scenarios:
- [Scenario: name — steps — expected result]

---

## Review
Date: ...
Verdict: PASS | NEEDS FIXES
...

---

## Verification
Date: ...
Summary: N passed, N failed
...

---

## Debug Fix
Date: ...
Bug: ...
Fix: ...
```

Phase status: `[ ]` pending · `[x]` done · `[!]` failed.
Parallel phases: `parallel-group: [A/B/C]` — same label = runs concurrently.

---

## Orchestrator Progress Output

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  RALPH-TEAMS  [N of M phases complete]  [SEQUENTIAL | PARALLEL]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Phase 1: ...          [done]         (haiku)
  ►  Phase 2: ...          [building...]  (sonnet) ┐ parallel-group A
  ►  Phase 3: ...          [building...]  (haiku)  ┘
  ○  Phase 4: ...          [pending]      (sonnet)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Status symbols: `✓` done · `►` building · `✗` failed · `○` pending

---

## marketplace.json Schema

```json
{
  "name": "ralph-teams-claude-plugin",
  "owner": { "name": "<github-handle>" },
  "metadata": { "description": "...", "version": "X.Y.Z" },
  "plugins": [{
    "name": "ralph-teams",
    "source": "./",
    "description": "...",
    "version": "X.Y.Z",
    "keywords": [...],
    "category": "productivity",
    "skills": "./skills/"
  }]
}
```

Both `metadata.version` and `plugins[].version` must match `plugin.json` version.
