# MVP boundaries

Read [`docs/mvp.md`](../../docs/mvp.md) for the rationale. This file is the checklist. Do not treat it as a second copy of the plan.

Before adding behavior, confirm it is inside the MVP section of the summary. If it is only in the deferred table, leave the hook and do not build the feature.

## Keep

- Clients send text to the proxy. Speech-to-text stays on the device. The proxy is the only caller of the model.
- One PWA. Google OAuth. The server does not store passwords.
- Local model and Gemini are a config switch. Prompt assembly does not fork per provider.
- Characters are universe-scoped. A claim is a single-winner transaction.
- Identity and claims live in the relational store. Time-varying facts are immutable events. Lore and major-event chronicles live in the vector store. The vector store is not queried for "who owned this at date T".
- Ownership and physical presence are different facts. Chance meetings are rolled in the proxy, then described by the model.
- Plot-armor and world-inertia prompts require an invented, coherent outcome and allow solutions the prompt did not list.
- Replies use speaker tags. Model calls are serialized. Injected context stays inside the budget in the summary.
- Events can be appended by code that is not the live turn, so a later simulator uses the same log.

## Leave out

Narrators and TTS, a background world simulator, a hard inventory ledger, synchronous co-op, encounter routines, owner-rescue notifications, native apps, and server-side transcription.

The compatibility hook for each one is in the deferred table of the summary. Implement the hook. Do not implement the feature.
