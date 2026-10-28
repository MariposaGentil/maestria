# narrative-engine

Asynchronous persistent narrative world. Players share a universe: a character belongs to the setting, not to one campaign, and can appear in someone else's session at another point on their own timeline. A game-master server assembles memory, lore, and time-filtered state, then asks a language model to continue the scene.

There is no application code in this repo yet. The design is the MVP plan.

## Read this first

- [Current status](CURRENT-STATUS.md) — what exists today, and choices settled after the plan.
- [MVP plan](docs/mvp.md) — product source of truth: architecture, data model, narrative rules, what is in the first version, and what is deferred on purpose.
- [Original conversation](docs/Arquitectura%20detrás%20de%20AI%20Dungeon.txt) — the design chat this plan compresses. Spanish. Use it when the plan is ambiguous.

## Shape

| Layer | Role |
| --- | --- |
| Inference | Text in, text out. A local 7B–8B model on an RTX 4060, or Gemini, chosen by configuration. |
| Proxy | FastAPI game master. Owns auth, the three stores, prompt assembly, and the presence rules. Clients talk only to this process. |
| Client | One installable web app. Dictation happens on the device; the server receives text. |

State is split on purpose. A relational database holds identity and character claims. An event log holds anything that depends on the in-game clock. A vector store holds lore and short chronicles, not the ledger.

## Agents

Agent guidance lives in [`.agents/`](.agents/README.md). [`AGENTS.md`](AGENTS.md) is only the entrypoint and points there. The product context those rules load is [`docs/mvp.md`](docs/mvp.md).
