# Run verbs in one shared namespace, not under lyx loom

`run-verbs-shared` · Note · enhancement, shed, loom

## Seen (operator, 2026-10-14)

Since `0850f76e` a darn run has a decision record, and `lyx loom decision add` works on it.
The operator finds it absurd that a `loom` command acts on something that is not a loom run.
The same holds for the other run verbs the orch uses on darn runs: `lyx loom status`, `goto`, `resume`, `approve`, `reject`, `start`, `review approve`.

## Direction (operator, 2026-10-14)

Darn and loom are both shed recipes and are to be handled the same way.
Verbs that act on any run move to one shared namespace, and `lyx loom` keeps only what is loom's own.
Not `lyx shed`: shed is only the FSM and knows nothing about decisions, reviews or approvals (operator).
For now the verbs stay under `lyx loom` (operator, 2026-10-14).

## Open

- The shared namespace's name, and which module owns it: the layer above shed that both recipes share (decision record, parent review, approve and reject) has no module of its own today; `internal/loomcli` serves darn runs too.
- Which verbs are recipe-neutral and which are loom's own.
- Every stencil, way-forward text and doc that names `lyx loom <verb>` for a darn run changes with it.
- Relates to `perch-budget-arbiter`, which gives darn and loom one config shape for review budgets.
