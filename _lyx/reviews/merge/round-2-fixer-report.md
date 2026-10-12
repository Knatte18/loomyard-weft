# Fixer-report, merge round 2

## Changes

- F1 (`state`): `keepUnknownInList` in `internal/state/keep.go` now pairs each new element with an old element that encodes as it, taken from the old elements not yet paired and preferring the same index, through the new `pairOldElement`.
  A shifted entry keeps its unknown keys; the doc comments say so.
  Tests: a table row group in the `pauserecord` `Update` test covers a live entry behind a lifted one and a resumed entry behind an expired one.
  The `state` test row that asserted the old drop ("removing entry 0 drops the unknown key…") asserted the wrong behavior, so it now expects the key kept.
- F2 (`pauserecord`): `internal/lock` removed from the allowlist and header comment in `leaf_enforcement_test.go`, which now calls `scankit.AssertImportAllowlistNoStale`; `doc.go` and `pattern/PATTERN-leaf-packages.md` state the standard library and `internal/state`, with `gofrs/flock` through it.

`lyx gate test ./internal/state ./internal/pauserecord` is green.

## Disputed

None.

## Deferred

None.
