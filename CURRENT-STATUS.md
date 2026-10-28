# Current status

Last updated: 2026-10-28

Public repository: https://github.com/MariposaGentil/maestria

Design only. No application code, schema, client, or running service.

The product spec is [`docs/mvp.md`](docs/mvp.md). This file records how far the work has gotten and which later choices are settled. It does not restate the plan.

## In the repo

- [`README.md`](README.md) — what the project is
- [`docs/mvp.md`](docs/mvp.md) — MVP plan
- [`docs/Arquitectura detrás de AI Dungeon.txt`](docs/Arquitectura%20detrás%20de%20AI%20Dungeon.txt) — source conversation
- [`.agents/`](.agents/README.md) — agent entry rules. [`AGENTS.md`](AGENTS.md) only points there

## Settled after the plan

These choices came from follow-up discussion. They are in force. The plan's open questions are still open.

- The game-master loop is the API proxy. The GPU machine only completes text. Both processes may share a PC.
- The loop is custom. General agent harnesses are not the core. Libraries in front of it are LiteLLM (or a thin OpenAI-compatible client), FastAPI, PostgreSQL, MongoDB, ChromaDB, and Google OAuth.
- SillyTavern is not the frontend, does not drive the databases, and its prompt builder is not embedded. It is a single-player chat app whose prompt code lives in its own screen.
- The proxy assembles the full prompt: world block (lore, event state, plot armor, presence), then the last few stored turns, then the player's action. The client never sees that prompt.
- [`fast-tavern`](https://github.com/echo744/fast-tavern) can rebuild SillyTavern-style cards inside Python. It is not a dependency. Revisit it only if SillyTavern character cards become a requirement.

## Not started

The first build step in the plan: a local model and Gemini behind a config switch, a FastAPI WebSocket, and one in-memory session. Storage interfaces get defined there even while persistence is still temporary.
