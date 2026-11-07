# Fixer-report, round 1

## Changes

- F1 (MEDIUM): fixed as proposed, commit `34a83e52`.
  `TestRenderClaudeSettings` gained rows for a scalar changed since, a user item in a heddle-owned list, no entry and no record, a blank file, a record holding no object, and an entry holding no object.
  The table struct gained `editBetween`, `recordOverwrite` and `invalidText`.
  The record-dropped-with-the-last-entry case is its own test, `TestRenderClaudeSettingsDropsTheRecordWithTheLastEntry`, since it needs three renders.
- F2 (LOW): fixed as proposed, commit `34a83e52`.
  `foldObjects` returns `invalid`; `renderObjectTarget` skips `dropStale` and records `mergeJSON(previous, folded)` while any entry fails to decode.
  Row `EntryHoldingNoObjectKeepsWhatTheLastRenderSet` covers it.
- F3 (LOW): fixed as proposed, commit `34a83e52`.
  The record is removed through the new `removeRecord` after the file's removal or write succeeds.
- F4 (LOW): fixed as proposed, commit `2b2489c2`.
  `TestEnsure_OwnWarpExcludeGainsTheSettingsFileOnce` seeds a hub `warp_exclude: A.md`, runs the reconcile, and expects the path appended, the marker tracked and a second walk to change nothing.

A gate finding then flagged a 120-character comment line in `settings_test.go`; the comment is now split at its colon into two lines, in a follow-up commit.

A second gate finding, the fabric-vocabulary guard, flagged bare `warp` tokens in the F4 test in `internal/hubreconcile`.
The test is renamed `TestEnsure_OwnExcludeListGainsTheSettingsFileOnce`, its fixture message drops the token, and it reads the key through `fabricengine.ExcludeListKey`, which was the unexported `warpExcludeKey`, now exported under a name free of the token.

F1 to F3 share one commit because they touch the same two files and the F1 rows exercise F2 and F3.

Gate: `lym gate test --upto 1 ./internal/heddleengine ./internal/hubreconcile` passed, with no failures.

## Self-check

- fix-code-correctness, LOW: no test pins the F3 order, since no row makes the file write or removal fail. Disputed, see below.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

- Self-check finding (fix-code-correctness, LOW), ground `every fix worse`: a test that pins the F3 order must make `writeRendered` or `os.Remove` fail.
  The portable way is a read-only directory, which does not fail when the tests run as root, so the test would pass or fail by environment.
  F3 asked for an order change, not a test.

## Deferred

Nothing deferred.
