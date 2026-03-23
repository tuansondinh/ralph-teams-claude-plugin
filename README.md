# ralph-teams

A Claude Code plugin that plans and builds features using sequential builder subagents (Haiku or Sonnet based on task complexity), automated E2E verification, an Opus code review pass, and manual verification with integrated debug.

## Quick Start

```
/teams:plan
```

That's it. Describe what you want to build — Claude handles the rest.

---

## Quick Install

Inside Claude Code, run:

```
/plugin marketplace add tuansondinh/ralph-teams-claude-plugin
/plugin install ralph-teams@ralph-teams-claude-plugin
```

Then run `/teams:plan` in any project.

---

## How it works

```mermaid
flowchart TD
    classDef user fill:#ffdfba,stroke:#ffb347,stroke-width:2px,color:#333
    classDef orch fill:#bae1ff,stroke:#5facff,stroke-width:2px,color:#333
    classDef agent fill:#baffc9,stroke:#42d669,stroke-width:2px,color:#333
    classDef doc fill:#ffffba,stroke:#e6e65a,stroke-width:2px,color:#333
    classDef cmd fill:#f3e8ff,stroke:#c084fc,stroke-width:2px,color:#333
    classDef optional fill:#ffe4e1,stroke:#ff9999,stroke-width:1px,stroke-dasharray:4,color:#333

    P1["/teams:plan — Discuss feature, write plan, get approval"]:::cmd
    P["ralph-teams/PLAN.md"]:::doc
    CX1["Codex second opinion on plan (optional)"]:::optional

    P1 --> P
    P --> CX1

    P2["/teams:run — Build each task sequentially"]:::cmd

    CX1 -->|"approved"| P2

    subgraph Build[" "]
        direction TB
        B1["Haiku/Sonnet Builder — Task 1"]:::agent
        B2["Haiku/Sonnet Builder — Task 2"]:::agent
        BN["Haiku/Sonnet Builder — Task N"]:::agent
        B1 --> B2 --> BN
    end

    P2 -->|"one fresh agent per task"| Build

    R["Opus Reviewer — Reviews all changes"]:::agent
    CX2["Codex second opinion on review (optional)"]:::optional
    REV["ralph-teams/REVIEW.md"]:::doc
    BF["Sonnet Builder — Applies blocking fixes"]:::agent
    DOCS["Haiku Scribe — Updates docs (optional)"]:::optional

    Build --> R
    R --> CX2
    CX2 --> REV
    REV --> BF
    BF --> DOCS

    P3["/teams-verify — Walk through scenarios manually"]:::cmd
    DBG["teams-debug — Fix bugs against the plan"]:::optional
    DOCS --> P3
    P3 --> DBG
```

Each task runs in its own isolated subagent with a clean 200k token context window. Results are committed after each task so you can always resume with `/teams:run`.

---

## Commands

| Command | What it does |
|---------|-------------|
| `/teams-plan` | Discuss → plan → optional AI review → approve → build → Opus review → fixes |
| `/teams-run` | Resume an existing plan from where it left off |
| `/teams-verify` | Walk through manual E2E verification scenario by scenario |
| `/teams-debug` | Fix a bug in relation to the active plan — usable anytime |
| `/teams-document` | Update existing docs (README, ARCHITECTURE.md, etc.) for the latest plan |

---

## Progress output

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  RALPH-TEAMS  Plan #3 — 2 of 4 tasks complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ✓  Task 1: Project Setup          [done]        (haiku)
  ✓  Task 2: Auth System            [done]        (sonnet)
  ►  Task 3: API Routes             [building...]  (sonnet)
  ○  Task 4: Frontend               [pending]      (haiku)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

`✓` done · `►` building · `✗` failed · `○` pending · `(haiku)` simple task · `(sonnet)` standard task

---

## Output files

All build artifacts are written to `./ralph-teams/` in your project:

| File | Contents |
|------|----------|
| `ralph-teams/PLAN.md` | Plan ID, tasks with complexity, acceptance criteria, verification scenarios |
| `ralph-teams/REVIEW.md` | Opus reviewer findings (blocking / non-blocking) |
| `ralph-teams/VERIFY.md` | Manual verification results |
