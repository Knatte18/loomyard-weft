# Webster round 1 — fixer report

Both findings were validated against the code and fixed, one commit each.
After each fix, `lyx gate test` passed for the packages it touched, at the unit tier and with `--tags integration`.

## Fixed

### F1 — the parked record's verb is spelled in three places

Commit `1fa3e4401`.
`internal/fabricengine/midmerge.go` now declares `MergeVerbMergeIn` and `MergeVerbMerge` beside `MidMergeState`.
`MergeIn` and `Merge` in `internal/fabricengine/merge.go` write the record with them.
The private `mergeInVerb` constants in `internal/landingshed/premerge.go` and `internal/loomcli/start.go` are deleted, and both packages, along with `start_midmerge_test.go`, compare against `fabricengine.MergeVerbMergeIn`.
The constant names avoid the fabric vocabulary words, since packages outside the vocabulary owner set read them.

### F2 — the "synced into the task" row does not reach the intersection

Commit `fc7cdf817`.
The premise is correct: the row merged the parent's weft branch into the task's, which moved the merge-base to the parent's tip, so `diff base..tip` and `diff parentTip..tip` were the same set.
The row is now "a parent change the task carried over is not reported".
It cherry-picks the parent's post-fork commit onto the pair's weft branch, which leaves the fork point where it was.
The fork-point diff therefore still lists the file, and only the parent-tip diff drops it, so the single-diff code would have reported it.
The `Base` special case for the merge is gone, and every row again asserts `Base` equals the fork point.
This corrects an existing row and adds no test.

## Disputed

None.

## Deferred

None.
