---
dispositions:
  - id: F1
    disposition: fixed
---

## Changes

- F1, commit `575d2c3c2`: in `internal/boardcli/verbs_test.go`, `stepListAndFindText` no longer compares `list --text`, `find --text` and `list --text --label bug` byte for byte.
  A new helper `textListingRows` returns the first line and each row's first field (handle).
  `list --text` is checked for the `prime owner: none` first line and handles `[T1 P0 E1]`, so the P0 root row still stands between the task and the note.
  `list --text --label bug` is checked for the same first line and handles `[T1]`, so the filter still drops the root row.
  `find --text later` is checked for exactly one row, handle `E1`, containing `later`.
  `stepListShowsASketch` now looks for an `S1` row containing `sketched` instead of pinning the `sketches:` line and the row's position; its doc comment follows.
  The sort, refusal and P0-ordering checks are unchanged.
- `lym gate test ./internal/boardcli` passed (106 passed, 0 failed).
- Gate repair, a separate commit: the verify run failed in `internal/battencli` tier1 at
  `TestBattenIntegration_GoneTaskWorktree/ARemovedPairBeforeLandFailsTheRunNamingTheCancel`.
  Its origin reconcile ran with no Bolt replayer registered, since the test binary never runs the root pre-run.
  The subtest now calls `boardkit.InstallReplay(t)` first, as the sibling integration tests do.
  `lym gate test --tier 1 ./internal/battencli` passed (150 passed, 0 failed).
  The failure was independent of F1's change, which touched only `internal/boardcli/verbs_test.go`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
