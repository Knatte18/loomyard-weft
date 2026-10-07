# Round 3 fixer report

All four findings fixed; nothing deferred.
No code was touched; every change is in `_lyx/plan/`, and the overview's `approved:` key is unchanged.

## Changed

**F1 — card 4's discarded `rebaseline.go` edit.**
`04-recorded-partition.md` now carries the `RebaselineDeps` change: `Batches` becomes `Active` and `Sizes`, `webstercli/rebaseline.go` passes `c.batcher` and `batcher.DiskSizes(c.geom.WorktreeRoot)`, and `Rebaseline` forms the batches with the new signature and refuses on `CheckBatchOrder`'s error.
Its Edit list gains `internal/websterengine/rebaseline.go`, `rebaseline_test.go` and `rebaseline_git_test.go`, and its ImpactSummary names `Rebaseline` among the `Batch` callers.
`07-rebaseline-tail.md` no longer introduces the deps, so card 7's edits to those files build on card 4's and discard none of them.

**F2 — overlapping `TestActive` rows.**
`06-batcher-profiles.md` merges the operator-only-profiles row into the no-such-profile row, which now also pins that the template's profiles are not filled into a present `profiles:` key.

**F3 — `BaseSHA` ordering and fixture wording.**
`03-calibration-report.md` defines earliest as the smallest `timestamp` on the envelope's `tool_result` line across every `webster` session file, and the `TestCountRun` fixture extends its existing `webster` session with a second file that sorts first and holds the later pair.

**F4 — untested tail order refusal.**
`07-rebaseline-tail.md`'s Tests gain one row in the existing refusal test: a tail holding a card that uses a later card's target refuses with `ErrBatchOrder` and leaves `State`, `Partition` included, unchanged.

## Deferred

None.
