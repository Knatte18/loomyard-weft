# Raddle releases the tasks it files, chains shared-package parts against the record order, and shows no dependencies

`raddle-order-and-deps-table` · Task · bug, loom, stencil

## Seen (operator and orch, 2026-11-07, raddle of `fabric-test-speed`)

- The Sketch's decision record (D9) said its `dotgit` worktree reader waits for nothing and was the largest single saving.
  Raddle's proposal put it fifth in the `internal/fabricengine` chain, behind three fixture moves, because the `packages-shared` check in `internal/raddleparser/check.go` refuses two unordered parts on one package.
  It named neither the departure from D9 nor the cost; the orch caught it, and only after a second round did raddle put the trade-off to the operator.
- The table raddle shows the operator in the pane for approval lists the parts with no dependencies, so the order of a split cannot be read from it.
- The operator approved the proposal to have its parts filed on the board, and raddle filed all 12 tasks of project `fabric-test-speed-work` released, not as drafts, so the dobby started the ones with no open dependency at once.
  Approving a proposal is not a release (operator, 2026-11-07).
  The operator had meant to run that project as a beam of its own; the root dobby started its tasks first, so they run from the root.

The stencil is `contracts/stencils/raddle/raddle-template-agent.md` (Step 3 orders parts "by what each takes from the others" and keeps unordered parts off one package); the proposal format is `contracts/specs/raddle-proposal-spec.md`.

## Wanted

- When parts share a package and must be chained, raddle orders the chain by the Sketch's own order first and, where the record names none, puts the part with the largest stated saving or the one others gain from earliest; a part the record says waits for nothing goes first in its chain.
- Every place the proposal's order departs from an order the decision record states is listed in the proposal and named to the operator with its cost, before approval.
- Raddle files every task it makes as a draft, project members included; a release stays the operator's, through `lym board set-status`, and the approval prompt says the tasks land as drafts, so a project can be made a beam before any of its tasks starts.
- The table shown to the operator for approval has a column of each part's dependencies, by part, so the order reads from the table alone.
