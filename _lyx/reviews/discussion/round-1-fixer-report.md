# Fixer report — discussion round 1

## Changes

- F1, fixed as proposed: D1's first bullet now requires X's status after the write to be not `done`; a sentence beside the labels sentence says X's status is the one the write leaves on it, keeping the relation symmetric in a batch; the rationale's done clause covers X as well as Y; Acceptance criteria's no-warning list gains "X is written with status done".
  Added Auto-mode assumption and support-log Question ledger entry Q12 for the pick.
- F2, fixed as proposed with the recommended pick: D1's status bullet excludes an abandoned Y, with a rationale sentence that a newly-ready start takes only tasks with no status (`readySlugs` in `internal/battencli/land.go`).
  Acceptance criteria's no-warning list gains "Y is abandoned"; the Auto-mode assumption names it; the Question ledger gains Q11.
- F3, fixed as proposed: D3's line now names every dependency that makes the pair siblings under D1, once, and states that a dependency already shared before the write is not named.

## Disputed

None.

## Deferred

None.
