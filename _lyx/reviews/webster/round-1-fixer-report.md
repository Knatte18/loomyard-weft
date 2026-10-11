# Webster round 1 — fixer report

## Changes

- **F1** (LOW, consistency), fixed as proposed with the first option, in commit `72bee26cc`.
  The premise held: `postBatchChecks` returned the restamp error after a successful `BindHandles` and before `logBindOutcomes`, a no-line case card 5 does not list.
  Card 5 lists four no-line cases and says that otherwise the pass logs `BindHandles`' outcomes.
  So the fix logs the trace and does not add a fifth case to `doc.go`.
  `internal/websterengine/recordbatch.go` now substitutes the `language-none` or `delta-unavailable` outcomes and logs the trace right after `BindHandles` returns without error, before `restampFingerprint` can return.
  The `doc.go` no-line list stays accurate unchanged.
  No test was added because the finding is not a coverage gap.
  `lyx gate test ./internal/websterengine/...` and `lyx gate test --tags integration ./internal/websterengine/...` pass, and so does `lyx loom lint-comments`.

## Disputed

None.

## Deferred

None.
