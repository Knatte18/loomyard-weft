# Fixer report — discussion round 2

Both premises were checked against the code before fixing:
`seatSpec` in `internal/seatengine/prompt.go` sets `KeepPane: !isChair` and `Timeout: table.Timeout` on every seat;
`Run` in `internal/seatengine/engine.go` starts the chair first, and only its `join` stops every started advisor;
`decideRound` in `internal/burlerengine/handoff.go` stops a half whose wait errored or that timed out.
Both premises hold.
`lyx loom validate-discussion` exits 0 after the fixes.

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  D3's early end now stops every started seat, the required advisor itself included, and names why: a timed-out or wait-errored run may still be live and an advisor's pane is kept.
  A seat it cannot stop there surfaces as `ErrSeatNotStopped` with its way forward.
  The D4 Acceptance criterion rows for the reviewer's death or timeout, its wait error and the audit or strict-parse error now say "chair and reviewer stopped", and the failed-seat-gate criterion stops every started seat, that advisor included.
- **F2 (MEDIUM), fixed as proposed, first option.**
  D10's predicate now counts only advisors whose run has not ended, timed out included.
  The deferral is capped by a new maximum-deferral constant in `shuttleengine`: a chair held past it notifies once regardless of advisor work.
  The Bound says why the table timeout cannot bound it.
  The hold-deferral Acceptance criterion gains both cases, the Auto-mode assumption covers both constants, and the support log's Rejected alternatives records the advisor-deadline bound.

## Disputed

None.

## Deferred

None.
