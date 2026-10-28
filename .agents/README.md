# Agent configuration

This folder is the vendor-neutral source of truth for agents in maestria. The root [`AGENTS.md`](../AGENTS.md) is the entrypoint and only points here.

Product context is not stored in this folder. Agents load it from the MVP summary, [`docs/mvp.md`](../docs/mvp.md). The original design chat is an archive, not a second spec.

## Structure

```text
.agents/
├── README.md
├── instructions/
│   ├── session-context.md    # Every session: read the MVP summary
│   ├── current-status.md     # Read and maintain ../CURRENT-STATUS.md
│   └── mvp-boundaries.md     # Non-negotiable constraints; details are in the summary
└── knowledge/
    └── INDEX.md              # Topic → section of docs/mvp.md
```

## How a session gets context

1. [`instructions/session-context.md`](instructions/session-context.md) sends the agent to [`docs/mvp.md`](../docs/mvp.md) in full, once per session.
2. [`instructions/current-status.md`](instructions/current-status.md) sends the agent to [`CURRENT-STATUS.md`](../CURRENT-STATUS.md), and requires an update in the same change when the project state moves.
3. [`knowledge/INDEX.md`](knowledge/INDEX.md) maps a later question to a section of that summary so the agent does not re-read the raw conversation.
4. [`instructions/mvp-boundaries.md`](instructions/mvp-boundaries.md) lists decisions that are easy to undo by accident. Each item points at the summary for the rationale.

When the summary and the transcript disagree, the summary wins. When implementation and the summary disagree, fix the implementation or update the summary in the same change — do not leave the drift in a chat.

## Adding guidance

Add an instruction file only when it changes what an agent does, and link it from the root `AGENTS.md`. Put new product facts in `docs/mvp.md`, then add a row to `knowledge/INDEX.md`. Do not paste the same fact into both places.
