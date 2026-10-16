# Fixer report — discussion, round 1

## Changes

- F1, fixed as proposed: D3 now runs the move as the first step of the reconcile every public engine op runs, before reconcile plans its dead-pane kill, untracked reap and binding clears, and says a dead leftover pane is moved before that kill can reach it.
  Premise checked against `internal/reedengine/reconcile.go`'s `planReconcile`, which puts every dead pane but the kept and exempt ones into its kill set.
- F2, fixed as proposed: D3 orders the Selvage heal before the move in the ops that heal (up and resume, per `internal/reedengine/lifecycle.go`, where `ensureSelvagePaneLocked` runs before reconcile), and the Bound now says the move never breaks out the shell window's last pane; such a pane waits for an up or resume to heal Selvage beside it.
  A new acceptance row covers a leftover strand pane that is the shell window's only pane, with Selvage missing.
- F3, fixed as proposed: D3 states the move is a standing step of every reconcile, keyed on a bound strand pane in the shell window and gated by no upgrade marker; "moving it once" left the Rationale.
- F4, fixed as proposed: the shuttle acceptance row now lists `below-parent`, `top`, `overview`, unknown and empty values as refusals that start no run.
- F5, fixed as proposed: the grep row names `below-parent`, `AnchorBelowParent` and `AnchorTop` with the same exception list.
  D1 also names the two retired constants, since the review noted no Decision removed `AnchorTop` by name.

## Disputed

None.

## Deferred

None.
