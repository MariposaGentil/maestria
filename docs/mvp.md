# MVP plan

Working name: **narrative-engine**. An asynchronous persistent narrative world: one shared universe per setting, played from a phone or browser, with the story model running on a local GPU or on Gemini.

This file is the product source of truth. It compresses the design conversation in [`Arquitectura detrás de AI Dungeon.txt`](Arquitectura%20detrás%20de%20AI%20Dungeon.txt). When the two disagree, this file wins. Open the transcript only to resolve a gap this summary does not cover.

The repo has no application code yet. Build only what this plan marks as MVP. Deferred items stay out of the first version, and the hooks listed with them stay in.

## What the player gets

A person signs in, picks a universe (for example D&D or Cyberpunk), and continues a campaign as a character who also exists for everyone else in that universe. Other players' characters can show up in the scene as NPCs, including at a different point on that character's timeline. What happens is stored and can change later sessions. The client is one installable web app. The player can type or dictate. The server answers with story text.

The product is a context-and-state engine around a language model. The model is interchangeable. Memory, claims, time, and presence are not.

## Decisions already made

| Topic | Choice |
| --- | --- |
| Model hosting | Local 7B–8B on the RTX 4060, and Gemini's free API. A config switch selects the engine. Prompt assembly stays the same. |
| Who calls the model | Only the proxy. Clients never hold the model URL or API key. |
| Speech to text | On the device (Web Speech API or the OS dictation engine). The server receives text. |
| Client | One PWA, installed from the browser, for phone and desktop. |
| Sign-in | Google OAuth. The server stores no passwords. |
| World scope | One universe per game. Characters belong to the universe, not to a campaign. |
| Character claim | One owner, enforced by a transaction. A claimed character cannot be claimed again. |
| Facts that change over in-game time | Immutable events in a document store. Not columns on the entity, and not vectors. |
| Lore | Vector store: static description, personality, and short narrative summaries of major events. |
| Presence | Separate from ownership. The proxy decides whether someone is physically there, including a dice roll for a chance meeting. The model does not decide that. |
| Harm to someone else's protagonist | The model must invent a coherent, reversible outcome. The prompt allows solutions it was not given. |
| Harm to a place that already has a later played future | World inertia: heavy damage is allowed; erasing the place is not. |
| Voices | Deferred. Every reply still uses speaker tags. |
| Scale target | One player, or a few people taking turns on one GPU. |

## Architecture

Three layers. The proxy can run on the same PC as the GPU, on another machine, or in the cloud. Even on the same machine, clients reach it over the network (localhost, a LAN address, or a tunnel), so a phone can play.

### Inference

The GPU machine is a text-in, text-out server (Ollama or LM Studio, OpenAI-compatible). It binds so the proxy can reach it. A remote proxy reaches it through a tunnel (Cloudflare Tunnel or ngrok).

Local models are 7B–8B GGUF at 4-bit (about 5–6 GB of the 8 GB on the RTX 4060), on the order of 30–45 tokens per second. Gemini Flash is the other engine: useful for planning a world, for long reasoning, and for post-session summaries so the GPU stays free for live turns. The free tier is enough for solo play (on the order of 15 requests per minute and 1,500 per day) and may use prompts for training, so prompts stay free of personal data. Local inference is the private path and the path without commercial content filters.

The switch is configuration (environment or a saved setting), not a code fork.

### Proxy

Python, FastAPI, WebSockets. This process is the game master:

- Validates the Google token and loads that player's campaign, character, and in-game clock.
- Reads the three stores.
- Decides presence and world-inertia before the model is called.
- Assembles the prompt.
- Calls the selected model and streams the reply.
- Writes significant outcomes as events, plus a short narrative summary for the vector store.
- Serializes model calls. One consumer GPU serves one generation at a time.

### Client

One progressive web app (manifest, service worker, HTTPS). Android and iOS install it from the browser; there is no store build in the MVP. The UI stays small: sign in, choose universe / campaign / character, play by text, dictate.

Dictation uses the Web Speech API. On a phone, a PWA delegates to the on-device recognizer (iOS `SFSpeechRecognizer`, Android `SpeechRecognizer`). Modern devices transcribe on the phone, including offline for installed language packs. The client sends text for a single utterance or for an ongoing conversation.

iOS PWAs need a tap before the microphone starts, and they are a poor host for background audio. That limit matters for later voice playback. The MVP does not play narrator audio.

## Data model

Three stores, queried together. Each one owns a different kind of truth.

### Relational store — identity and claims

PostgreSQL. SQLite is acceptable for the first local prototype if the proxy talks to it through the same repository interface.

Holds records that must exist exactly once and that do not depend on the in-game clock:

- Users (Google subject id).
- Universes.
- Campaigns and the player's current in-game time.
- Characters: id, universe, `is_claimed`, `owner_id`.
- Locations, factions, and unique objects: id, universe, name, and stable facts such as region. Not the current owner, not "destroyed", not "on fire".

A claim is a transaction. If two players claim the same character, one commit wins.

### Event store — the timeline

MongoDB, one immutable JSON document per change. The document has an entity id, a universe id, an in-game timestamp, an event type, and a payload that may differ by type (a purchase has a buyer and a price; a fire has damage and a cause).

The relational row says the club exists. The event log says who held it, and in what condition, at a given in-game date. The proxy rebuilds state by reading events with `timestamp <= player_time`, in order. Events after that instant are invisible to that player.

Appending an event is the only way the world changes. A later world simulator, a human, or the live turn all write the same kind of document.

### Vector store — lore

ChromaDB (local). Static description, personality, ideology, and a narrative summary of a major event. Major-event summaries carry the in-game time so retrieval can drop the future.

The vector store is not the ledger. Similarity search will surface a vivid future event and skip a dull past one, it will not return a full ordered history, and embedding every minor action ("opened a door") drowns the lore. Those facts stay in the event log. When something large happens, the model writes a short chronicle and that chronicle is what gets embedded.

### What is an entity

Anything that can be found by another session and that can change: characters, locations, factions and companies, unique objects, and global events. Ordinary pocket clutter stays in the scene summary for the MVP. A unique sword that another player should be able to find later is an entity plus events.

## Turn pipeline

1. The client sends text and the active campaign id, with the Google token on the WebSocket.
2. The proxy loads the campaign clock and the entities in play (the place the player entered, the player character, named targets).
3. It rebuilds each entity's state from events at or before that clock.
4. It separates ownership from presence. The owner of the club is a fact. Whether that person is in the room is a different fact (`last known location` at this clock, or a presence roll).
5. It retrieves lore only for entities that passed the time filter. Deep history is retrieved when the player investigates the past, not on every turn.
6. It injects plot armor or world inertia when another player's protagonist, or a place with a later recorded future, is at risk.
7. It sends a small prompt: system rules, relevant lore, the time-filtered state, a short summary of older play, the last few turns, and the player's action rewritten into story form when needed (a move command becomes narrative continuation, not a raw chat line).
8. It streams the reply, which uses speaker tags.
9. Significant outcomes become events. A major outcome also becomes a short lore summary.

Starting context budget, to be tuned rather than treated as a law: the last four turns, and historical summaries of at most about 200 words. Past that, an 8 GB card starts swapping and generation collapses, well before the model's theoretical context length. Stay near 8,000–12,000 tokens.

## Narrative rules

**Shared history.** A character can appear in a session their owner is not playing, including at a time that owner has not played yet. The outcome is appended to that character's history and can affect other campaigns.

**Plot armor.** When a claimed character is in a session their owner is not playing, the prompt states that they cannot die or suffer an irreversible change, and that the model must invent a universe-consistent way out. The instruction explicitly allows solutions that are not listed (a third party, a malfunction, the environment, or anything else that keeps the scene coherent). Listing only "falls into a coma" produces the same rescue every time.

When the owner is the one playing, the player decides what the character attempts. The model narrates the world and the consequences. When the character is unclaimed, the model plays them from their lore.

**World inertia.** The same idea applies to places and other entities that already have events in the future of the active clock. A player in 2025 does not get to delete a club that another player has already used in 2030. The model can burn it, close it, or damage it. It cannot remove it from the later timeline.

**Ownership is not presence.** Knowing that character A owns club X must not make the model put A behind the bar. If A's location at this clock is elsewhere, the prompt says A is the owner and is absent. The model may show A's taste in the room and in what staff say. It may not walk A on stage unless the player does something that would actually summon the owner.

**Chance meetings are a dice roll.** If A could plausibly be there, the proxy rolls before the model is called. A miss injects the absent-owner directive. A hit injects an active-presence directive: A is in the scene as part of the background (a corner, a back room, a conversation already in progress), the player chooses whether to approach, and plot armor still applies. The model does not roll this itself. A later version can weight the roll from a routine ("here on Friday nights"). The MVP uses a single configured probability.

**Speaker tags.** Replies name the speaker, for example `[Narrator]:` and `[Barkeep]:`. The MVP only displays that text. A later voice step can map each tag to a voice without changing the story engine.

## Authentication and sessions

The PWA offers Google sign-in and sends the token on every WebSocket connection. FastAPI checks the token and loads campaigns for that subject id. Firebase Authentication is the shortest client implementation; the requirement is Google OAuth and a verified subject id, not a particular client SDK.

A campaign is a meeting point: pointers to universe entities, plus that playthrough's clock and transcript. It does not own the characters.

## Deferred, and the hook that stays

| Deferred | Why it waits | Hook the MVP keeps |
| --- | --- | --- |
| Narrators and TTS (Edge-TTS, Piper, XTTS) | Extra systems and, for local neural voices, VRAM the 8 GB card does not have. | Speaker tags on every reply. Streaming replies so a later step can speak a finished sentence while the model continues. |
| World simulator (decay, factions moving while nobody plays) | Needs scheduled jobs and careful writes. | Any writer may append an event. Neglect, a sale, or a fire is a later event, not a new schema. |
| Hard inventory, gold, ammunition | Models are bad at arithmetic, but a full ledger is a second product. | Entity tables already exist. Unique objects can be entities now. Common gear stays in the scene summary. |
| Synchronous co-op | Two people in one scene need conflict rules the timeline does not have yet. | WebSockets are already the session transport. Later this is a room fan-out. |
| Routines that change encounter odds by in-game hour | Needs a schedule on the character. | The presence roll is already a proxy decision with a probability input. |
| Notice to the owner when plot armor saved their character | Raised and not decided. | The rescue is already an event on that character, so a later notifier has a record to read. |
| Native iOS and Android apps | One web client is the MVP. | The API is the contract. A native client would send the same text and token. |
| Server-side transcription (Whisper) | Spends VRAM and bandwidth the client path avoids. | The server accepts text only. |

## Suggested build order

One person, roughly 8–10 hours a week. This is a sequence, not a deadline.

1. **Loop.** Ollama or LM Studio, FastAPI WebSocket, config switch to Gemini, one in-memory session. Define the turn pipeline as an interface even while storage is temporary.
2. **Identity.** Google OAuth, a user, a universe, a campaign, a character pointer, an in-game clock.
3. **Ledger.** Relational entities and the claim transaction. Event documents and "state at time T". Tests that a 2034 event is invisible to a player in 2028.
4. **Injection.** Lore retrieval with the same time filter. Plot armor, world inertia, absent-owner text, and the presence roll. Speaker tags.
5. **PWA.** Installable shell, sign-in, campaign picker, text play, dictation, streamed reply.
6. **Calibration.** Prompt tests: the model does not kill a protected character, does not summon an absent owner, and does not describe a future event.

## Risks

- **Time is the hard part.** Paradoxes are a data problem. Plot armor and world inertia are the MVP answers; they need prompt tests, because an 8B model will drop a long instruction.
- **Context on 8 GB.** The proxy must truncate. Sending the whole chronicle will OOM or crawl.
- **One GPU is a queue.** Fine for solo or turn-taking. Ten simultaneous live sessions need more GPUs or Gemini. Do not build that scaler in the MVP; do serialize calls so overlapping requests fail cleanly instead of corrupting a generation.
- **iOS PWA limits.** Plan the microphone behind a button. Do not depend on audio continuing in the background.

## Open questions

- Product name.
- Exact JSON fields for each event type. The shape must stay filterable by entity, universe, and in-game time, and readable by the model after the proxy projects it.
- Whether the owner's "your character escaped elsewhere" notification ships, and how it is worded.
- Default chance-meeting probability per archetype (recluse versus regular). The MVP needs a number in config; the archetypes can wait.
