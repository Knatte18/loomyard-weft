# Intake reads the hub own repository, selfreport stays LoomYard-only, and the orch runs intake only on the operator word

`intake-own-repo` · Task · bug, board, orch, selfreport, stencil

## Seen (orch, 2026-10-21)

The quarry hub's new orch ran `lyx board intake` on its own, as its role tells every orch at every wake and landing (`contracts/stencils/orch/orch-template-role.md`), and imported LoomYard issues #783 and #785 onto the quarry board.
`lyx board intake` reads the selfreport inbox, which `internal/selfreportengine/selfreport.go` hard-codes as `Knatte18/loomyard`, whatever hub it runs in.

## Operator decisions (2026-10-21)

- Selfreport and intake are two different things.
  - Selfreport reports LoomYard bugs, and only those, to `Knatte18/loomyard`, from whatever hub lyx runs in. It never reports anything about another repository.
  - `lyx board intake` takes in every GitHub issue of the repository the hub works on: the hub's own warp repository, read from its binding or origin, never a hard-coded repository. For the quarry hub that is `Knatte18/quarry`; only the loomyard hub's intake reads `Knatte18/loomyard`.
- Intake never goes through selfreport: it does not call `internal/selfreportengine`, and its GitHub side (listing, reading, commenting on and closing the hub repository's issues) lives apart from selfreport's.
- A bug or other issue anyone finds in another repository, such as quarry, is filed to that repository's own issues, never through selfreport.
- The operator controls intake: the orch role no longer runs `lyx board intake list` at wakes and landings, nor starts an intake fork on its own. The orch runs intake only when the operator asks, and the intake fork procedure stays in the role for that case.

## Clean-up in this task

- `internal/selfreportengine/inbox.go` and the issue reads it serves intake with move out of selfreport.
- Every place that says or assumes intake reads the selfreport inbox is updated: `internal/boardcli`, `internal/boardengine` and `internal/selfreportengine` docs, the `ly:board` skill, and the orch role stencil.
