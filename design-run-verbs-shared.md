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

## Open

- The name of today's variant, since `loom` now names the family: the board's `recipe` field says `loom` for it today, and an empty `recipe` means it.
- How `loom` as the general driver relates to `shed`: `lym shed` already carries `run`, `step`, `status`, `pause` and `goto`, which the general `lym loom` verbs would overlap.
- Which packages take the general name: `internal/loomcli` already serves darn runs, while the packages and docs that mean today's variant need the new name.
- Every stencil, way-forward text and doc that says "loom" for the variant changes with it.
