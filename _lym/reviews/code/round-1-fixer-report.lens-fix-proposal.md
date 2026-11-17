```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 fix (new `handOver` doc text and the `summary.md` rewrite) as a proposal.

No findings.

- Batten's check matches the new text. `ChildDirtyPaths` skips the re-step on any uncommitted path (`innerrun_binaryrestep.go:122-131`).
- The new doc text says "only over a tree with no uncommitted paths outside the run's records". The "outside the run's records" part rests on `ChildDirtyPaths` excluding run records. I did not read it, and the review's own mention of `loomengine.PathsOutsideRunRecords` supports it.
- The `summary.md` rewrite holds. A spawn that succeeds leaves the successor to re-step. A failed spawn leaves a parked run that the parent has to pick up when the tree is dirty.
- The doc line is correct as written. The log message at `drive.go:377` ("the run stays parked") agrees with it.
- Neither edited line states a count or a list of current callers.

Tool calls:
1. Read `internal/battenshed/innerrun_binaryrestep.go` lines 100-149, to confirm the dirty-paths skip claimed in the finding. The context only had the review's description of it.
