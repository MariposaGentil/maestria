# AGENTS.md

Entrypoint for agents in this repo. The canonical guidance is [`.agents/`](.agents/README.md). Read that tree; do not copy it.

Provider-specific files (`.cursor/`, `CLAUDE.md`, `.github/copilot-instructions.md`, and similar) may only point at `.agents/`. They must not restate the product, the MVP, or these instructions.

## Load order

1. Read this file.
2. Read [`.agents/instructions/session-context.md`](.agents/instructions/session-context.md) and follow it before designing, scoping, or editing anything.
3. Read [`.agents/instructions/current-status.md`](.agents/instructions/current-status.md) and [`CURRENT-STATUS.md`](CURRENT-STATUS.md). Follow the instruction again before finishing any task that changes project state.
4. Read [`.agents/instructions/mvp-boundaries.md`](.agents/instructions/mvp-boundaries.md) before proposing or writing behavior.
5. After the session-context read, use [`.agents/knowledge/INDEX.md`](.agents/knowledge/INDEX.md) to reopen only the part of the summary the task needs.

## Instructions

- [`session-context.md`](.agents/instructions/session-context.md) — where product context comes from, and which file is authoritative.
- [`current-status.md`](.agents/instructions/current-status.md) — keep [`CURRENT-STATUS.md`](CURRENT-STATUS.md) matched to what exists and what has been settled.
- [`mvp-boundaries.md`](.agents/instructions/mvp-boundaries.md) — constraints that keep the first version compatible with later work. Detail lives in the summary, not here.
