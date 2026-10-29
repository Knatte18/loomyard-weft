# loom names the general run; today's loom is one variant

`run-verbs-shared` · Note · enhancement, shed, loom

## Seen (operator, 2026-10-14)

Since `0850f76e` a darn run has a decision record, and `lym loom decision add` works on it.
The operator finds it absurd that a `loom` command acts on something that is not a loom run.
The same holds for the other run verbs the orch uses on darn runs: `lym loom status`, `goto`, `resume`, `approve`, `reject`, `start`, `review approve`.
A third recipe is planned, `Hardener` (the note `hardener`), which would make three recipes steered with `lym loom`.

## Direction (operator, 2026-10-29)

`loom` becomes the name of the general run in Shed, the generic driver of a run, and `lym loom ...` stays as it is today.
`loom` is the name of the hub itself, loomyard, so it fits the general run.
What is called loom today is one variant of it, darn is another, and `Hardener` becomes a third.
This replaces moving the run verbs to a new shared namespace.

Today's variant is renamed `tapestry` (operator, 2026-10-29): a picture woven to a full-size drawing made first, which is the discuss-and-plan-before-building path.
`weave` was rejected for sitting too close to `Webster`, who weaves.

## Staging (operator, 2026-10-29)

Two steps, so that no search has to tell the family from the variant:

1. A mechanical rename of today's variant from `loom` to `tapestry`, done by a Go program under `tools/` like the `lyx` to `lym` rename (`ef912d41`, `tools/wordswap`).
   The `lym loom ...` verbs stay as they are, since they become the general ones.
2. Move the generic parts back under `loom`, as design work, once every remaining `loom` is a candidate for the family.

## Open

- Which packages, docs and stencils are the variant's and which are general; the rename program's inclusion and exclusion lists follow from it.
- What the board's `recipe` field and an empty `recipe` mean during step 1, since existing entries and runs carry `loom` or nothing.
- The strand names `ly:LOOM` and the texts that name them.
- How `loom` as the general driver relates to `shed`: `lym shed` already carries `run`, `step`, `status`, `pause` and `goto`, which the general `lym loom` verbs would overlap.
- Which packages take the general name: `internal/loomcli` already serves darn runs, while the packages and docs that mean today's variant need the new name.
- Every stencil, way-forward text and doc that says "loom" for the variant changes with it.
