# Fixer-report — webster round 1

## Changes

- **F1** (fixed as proposed): `closeAlignment` in `internal/parentreview/gate.go` calls `Aligner.Close` only when `c.cfg.Aligner != nil` and settles the record through `Store.SettleAlignment` either way.
  A subtest in `TestGate_Alignment` (`internal/parentreview/alignment_test.go`) rebuilds the gate with no `Aligner` over a running record and asserts a parent verdict, the wait bound and `final()` each settle it as stopped without a panic.
- **F2** (fixed as proposed): `internal/loomengine/template.yaml` has two spaces before the `#` on the `parent_review_wait_min` line again.

`lyx gate test ./internal/parentreview ./internal/loomengine` is green.

## Disputed

None.

## Deferred

None.
