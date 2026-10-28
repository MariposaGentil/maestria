# Session context

Apply at the start of every session, before exploring the repo or proposing a design.

## Source of truth

Read [`docs/mvp.md`](../../docs/mvp.md) from start to finish. That summary is the product context for this repo: what the MVP includes, how the three layers and three stores divide the work, the narrative rules, the deferred list, and the open questions.

Do not reconstruct the product from the raw conversation, from memory of a previous session, or by guessing ahead of the code. The transcript, [`docs/Arquitectura detrás de AI Dungeon.txt`](../../docs/Arquitectura%20detrás%20de%20AI%20Dungeon.txt), is the archive the summary was written from. Open it only when the summary is silent on the question. If you use it, say so, and prefer updating the summary over leaving a new decision only in chat.

## After the first read

For a follow-up question in the same session, open [`../knowledge/INDEX.md`](../knowledge/INDEX.md) and read the linked section of `docs/mvp.md`. Do not open every section again, and do not open the transcript because a keyword appeared there.

## When the task changes the product

- A behavior marked deferred in the summary stays deferred unless the user explicitly pulls it into scope.
- A new product decision gets written into `docs/mvp.md` in the same change, and the index row is updated if the topic is new.
- Open questions at the bottom of the summary are undecided. Do not pick one silently and build it.

## What this repo is

Design only, until code exists. There is no runtime, schema, or client to infer the product from. If `docs/mvp.md` and the code later diverge, trust the summary until the user says the code is the correction, then update the summary to match.
