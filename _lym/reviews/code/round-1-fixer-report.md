# Burler fixer-report, code review round 1

## Changes

- **F1** (LOW), fixed as proposed, commit `8cfc3701`.
  `TestDriftAwait` in `internal/driftengine/await_test.go` gains two rows, both expecting the plain capture and defer way forward and no `--scope`.
  One has a failed record with `Skip` set and `WroteNoFile` false (reason "exit 3").
  The other has `WroteNoFile` true and no `Skip`.
  The review proposed the second row under partial scope; it runs under full scope, since the row pins the empty skip list and the scope does not change the gate.
- **F2** (LOW), fixed as proposed, commit `3819be91`.
  `TestRunCapture` in `internal/driftengine/capture_test.go` gains a one-skipped-test row expecting "scope partial skipped 1 test,".
- **F3** (NIT), fixed as proposed, commit `3819be91`.
  `noFileReason` in `internal/driftengine/capture.go` counts `len(slices.Compact(slices.Sorted(slices.Values(skip))))` for the singular check and the printed count.
  A row with a repeated skip pins the distinct count.
- F2 and F3 share one commit because both edit `capture_test.go` and the tools here take no interactive hunk staging.

Gate: `lym gate test ./internal/driftengine` passed at tier 0 and at `--tier 1`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW: the F3 dedupe is a no-op on production paths, since every production caller builds its skip list through `SkipList`, which already deduplicates. Disposition: fixed as proposed, kept.
  `RunCapture` takes the list as told input and does not promise a deduplicated one, so the reason's count stays correct whatever a caller passes, and the change is harmless.

## Disputed

None.

## Deferred

None.
