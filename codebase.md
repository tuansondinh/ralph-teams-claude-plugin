# ralph-teams-plugin — Codebase Reference

**Version:** 1.2.1 | **Type:** Claude Code Plugin (Markdown-based, no compiled code)

---

## File Map

```
ralph-teams-plugin/
├── .claude-plugin/
│   └── marketplace.json          # Plugin registry metadata
├── agents/
│   ├── teams-builder.md          # Builder agent definition
│   └── teams-validator.md        # Validator agent definition
├── skills/
│   ├── plan/SKILL.md             # /teams:plan — discuss → plan → execute (single plan)
│   ├── run/SKILL.md              # /teams:run  — resume a single plan
│   ├── loop-plan/SKILL.md        # /teams:loop-plan — phased plan + execute
│   └── loop-run/SKILL.md         # /teams:loop-run  — resume a phased plan
├── README.md
├── FLOW.md                       # Mermaid execution flow diagram
└── codebase.md                   # This file
```

---

## Plugin Architecture

All files are **Markdown with YAML frontmatter** — no build step, no package manager.

### Agent Files (`agents/`)

Frontmatter fields: `name`, `description`, `model` (sonnet/opus/haiku).

| Agent | Name | Role |
|-------|------|------|
| `teams-builder.md` | `teams-builder` | Implements tasks, commits, messages Validator |
| `teams-validator.md` | `teams-validator` | Reviews commits, returns PASS/FAIL, handles pushbacks |

### Skill Files (`skills/`)

Frontmatter fields: `name`, `description`, `user-invocable: true`.

| Skill file | Command | Purpose |
|------------|---------|---------|
| `plan/SKILL.md` | `/teams:plan` | 6-step flow: discuss → plan → E2E requirements → optional AI review → approve → execute |
| `run/SKILL.md` | `/teams:run` | Resume a single plan; spawns fresh Builder+Validator for remaining tasks |
| `loop-plan/SKILL.md` | `/teams:loop-plan` | 5-step flow: discuss → phased plan (Group numbers) → E2E → approve + mode → execute |
| `loop-run/SKILL.md` | `/teams:loop-run` | Resume a phased plan; respects Mode: sequential/parallel and Group dependencies |

---

## Execution Model

```
Orchestrator (plan or run skill)
  └── TeamCreate → spawns teammates
        ├── teams-builder   (implements tasks)
        └── teams-validator (reviews commits)
              ↑
              └── Communicate peer-to-peer via `message` tool (NOT through orchestrator)
```

**Single-plan mode** (`plan` / `run`):
- One shared task list; Builder claims tasks sequentially
- Validator reviews each commit; max 2 pushbacks per task
- Orchestrator monitors and surfaces progress to user

**Phased mode** (`loop-plan` / `loop-run`):
- Tasks grouped into **Phases**; phases grouped into **Groups** (dependency layers)
- `Mode: sequential` — one phase at a time in order
- `Mode: parallel` — all phases in the same Group run simultaneously
- Separate Builder+Validator pair per phase

---

## Key Contracts

### Builder workflow
1. Read `.build/PLAN.md` for acceptance criteria
2. Implement one task, commit, run `git rev-parse HEAD`
3. Send commit SHA + summary to Validator via `message` tool
4. On `FAIL`: fix, re-commit, re-send
5. On `PASS`: mark task "completed", claim next task

### Validator workflow
1. **Standby**: wait for Builder `message`
2. On message: `git log --oneline -5` + `git diff HEAD~1..HEAD --stat`
3. Code-review against acceptance criteria; run E2E if required
4. Reply via `message`: `VERDICT: PASS` or `VERDICT: FAIL` with specific blocking findings
5. Never fix code; only review

### PLAN.md format
Located at `.build/PLAN.md`. Written by the orchestrator skill.

```markdown
# Teams Plan: [Feature Name]
Generated: [date]
Status: approved
Mode: single | sequential | parallel

## Tasks           ← single-plan mode
- [ ] Task 1
- [x] Task 2 (completed)

## Acceptance Criteria
- ...

## E2E Testing Requirements
...

---
### Phase 1: [Name]   ← loop-plan mode
Group: 1
Status: pending | in-progress | done | partial | failed
Goal: ...
Tasks:
- [ ] Task A
Acceptance Criteria:
- ...
```

---

## Orchestrator Progress Output

```
► Task [N]: Building...
► Task [N]: Validating...
⟲ Task [N]: Pushback received. Retrying...
✓ Task [N]: Complete!
```

---

## marketplace.json Schema

```json
{
  "name": "ralph-teams-claude-plugin",
  "owner": { "name": "<github-handle>" },
  "metadata": { "description": "...", "version": "X.Y.Z" },
  "plugins": [{
    "name": "teams",
    "source": "./",
    "description": "...",
    "version": "X.Y.Z",
    "keywords": [...],
    "category": "productivity",
    "skills": "./skills/"
  }]
}
```

Plugin `version` must always match `metadata.version`.
