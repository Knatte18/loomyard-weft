```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency: checked the F8 change set (`RangeDiff` seam, range key over the full diff) against its siblings, twins and cross-references.

**LOW**: `internal/driftengine/verify_test.go:1477`, the nil-`ReviewBase` "no range read" assertion, was left unchanged.
- The assertion is `len(f.changedFilesBases)+len(f.commitMessagesBases) != 0`, with the message "want one judge for the cell alone and no range read".
- The fix added a third range read, `RangeDiff`, with its own `rangeDiffBases` record.
- It extended the twin base assertion in the first subtest ("want the review base alone") to include `rangeDiffBases`, but not this one.
- A `RangeDiff` call made without a review base would pass this case.

The rest agrees:
- The seam lists (`verify.go`, `shedrecipe/entries_drift.go`, its test, `envkit`, `battencli`, `driftwiring.go`) all carry `RangeDiff`.
- The prose that claims the re-judge on a change to the range now matches the code: the `doc.go` reuse line, the `records.go` `RangeKey` comment, the `checkAndJudge` and `uncausedReason` comments, `docs/overview.md` and the judge stencil.
- No other doc outside `_lym` mentions `range_key` or the old path-and-diff key.

Tool calls:
1. One grep for `range_key`, the old key wording, `CommittedFileDiffs` and the seam names across the tree: my context lacked a repo-wide search for docs or twins outside the files the fixer read.
