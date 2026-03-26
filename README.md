# ralph-teams

Agent team skills for planning and building features with sequential builder subagents, automated E2E verification, and a code review pass.

**Version:** see [VERSION](./VERSION)

## Runtimes

| Folder | Runtime | Install |
|--------|---------|---------|
| [`claude/`](./claude/) | Claude Code (Claude Code plugin) | Install via Claude Code plugin manager, source: `./claude/` |
| [`codex/`](./codex/) | Codex CLI (`gpt-5.4`) | `npm install` from `./codex/` |

## What it does

1. **/teams-plan** — discuss a feature, generate a phased plan, execute with builder subagents, Opus/gpt-5.4 review pass
2. **/teams-run** — resume an existing plan
3. **/teams-verify** — walk through manual E2E verification
4. **/teams-document** — update project docs
5. **/teams-debug** — fix a bug against the active plan

## Versioning

Both runtimes share the same version number. Bump `VERSION` and both `claude/.claude-plugin/` and `codex/package.json` when releasing.
