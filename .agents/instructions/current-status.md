# Current status

Apply on every session, and again before finishing a task that changes the state of the project.

## Read

Read [`CURRENT-STATUS.md`](../../CURRENT-STATUS.md) after [`docs/mvp.md`](../../docs/mvp.md). The plan says what the product is. The status file says what exists today and which follow-up choices are already settled. Do not re-open a settled choice unless the user does.

## Update

Update `CURRENT-STATUS.md` in the same change when any of these happen:

- Code, schema, config, or a client appears or is removed
- A plan step starts or finishes
- A dependency is adopted or dropped
- A follow-up discussion settles or reverses a choice
- The next concrete step changes

Leave the file alone for a question that does not change any of those.

## Shape

Keep the existing sections. Edit them in place.

- `Last updated` — the day of the edit (`YYYY-MM-DD`)
- `In the repo` — paths that exist, one line each
- `Settled after the plan` — choices made after [`docs/mvp.md`](../../docs/mvp.md), in force now
- `Not started` — the next concrete step, or replace it with what is in progress

Put product rules in `docs/mvp.md`, not here. A one-line status entry may point at the plan. Do not copy a section of the plan into this file.
