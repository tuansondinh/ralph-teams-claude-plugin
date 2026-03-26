# ralph-teams

Agent team skills for planning and building features with sequential builder subagents, automated E2E verification, and a code review pass.

**Version:** see [VERSION](./VERSION)

---

## Runtimes

| Folder | Runtime | Models |
|--------|---------|--------|
| [`claude/`](./claude/) | Claude Code plugin | Sonnet (builder) · Opus (reviewer) · Haiku (simple phases) |
| [`codex/`](./codex/) | Codex CLI | gpt-5.4 (builder + reviewer) · gpt-5.4-mini (simple phases) |

---

## Installation

### Claude Code Plugin

> **Note:** The plugin root is `claude/` — point your installer there, not the repo root.

**From GitHub (recommended):**
```
github:tuansondinh/ralph-teams-claude-plugin/claude
```

**Local install:**
```bash
# In your Claude Code settings, add the plugin pointing to the claude/ subfolder:
/path/to/ralph-teams-claude-plugin/claude
```

If you previously had this plugin installed pointing to the repo root, update your path to include `/claude`.

---

### Codex CLI Plugin

> **Note:** Run all commands from inside the `codex/` subfolder.

**Local install:**
```bash
cd codex/
npm install -g .
```

**Or link for development:**
```bash
cd codex/
npm link
```

---

## Skills

| Command | What it does |
|---------|-------------|
| `/teams-plan` | Discuss a feature, generate a phased plan, execute with builder subagents, then Opus review |
| `/teams-run` | Resume an existing plan |
| `/teams-verify` | Walk through manual E2E verification |
| `/teams-document` | Update project docs (README, ARCHITECTURE.md, etc.) |
| `/teams-debug` | Fix a bug against the active plan |

---

## Versioning

Both runtimes share the same version number (see `VERSION`). When releasing:
1. Bump `VERSION`
2. Update `claude/.claude-plugin/plugin.json` and `claude/.claude-plugin/marketplace.json`
3. Update `codex/package.json`
4. Commit + push
