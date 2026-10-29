# Run verbs in one shared namespace, not under lym loom

`run-verbs-shared` · Note · enhancement, shed, loom

## Seen (operator, 2026-10-14)

Since `0850f76e` a darn run has a decision record, and `lym loom decision add` works on it.
The operator finds it absurd that a `loom` command acts on something that is not a loom run.
The same holds for the other run verbs the orch uses on darn runs: `lym loom status`, `goto`, `resume`, `approve`, `reject`, `start`, `review approve`.

A third recipe is planned: `Hardener`, a `Shed` wrapping `Tenter` (the note `hardener`).
With loom, darn and Hardener, three recipes of one kind would all be steered with `lym loom` (operator, 2026-10-29), which sharpens the point.

## Direction (operator, 2026-10-14)

Darn, loom and Hardener are shed recipes and are to be handled the same way.
Verbs that act on any run move to one shared namespace, and `lym loom` keeps only what is loom's own.
Not `lym shed`: shed is only the FSM and knows nothing about decisions, reviews or approvals (operator).
For now the verbs stay under `lym loom` (operator, 2026-10-14).

## Open

- The shared namespace's name, and which module owns it: the layer above shed that every recipe shares (the decision record) has no module of its own today; `internal/loomcli` serves darn runs too.
- Which verbs are recipe-neutral and which are loom's own.
- Every stencil, way-forward text and doc that names `lym loom <verb>` for a darn run changes with it.
