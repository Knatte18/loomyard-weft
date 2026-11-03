```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Unchecked findings: none.

- **F1:** all four helpers named in the review now have a split-root case. `trackedRel` and `suspectBlobs` are covered in `suspect_test.go`, `writtenWorktreePaths` in `runwrites_test.go`, and `failBatch` in `batchfail_integration_test.go`.
- **F1, extra callers:** the callers `findingtext.go:117`, `reset.go:436` and `recoverbatch.go:212` use the same helpers with the same root pair. The new cases pin the helpers themselves, so those callers need no case of their own.
- **F2:** the fixture's `Root: backend` now covers every verb the fixture drives. The `DiskSizes` sites (`cli.go:292`, `runlevel.go:511`, `rebaseline.go:110`) read `WorktreeRoot`. The begin-batch and rebaseline tests run through them with the rooted plan, and the gate passed.
- **Coverage limit:** `DiskSizes` takes a root and a card's paths, and the fixture's root-prefixed Create target does not exist on disk. A swap back to the anchor would therefore change only the size estimate, and no assertion reads that estimate. This is a limit of the reviewer's proposed fix, not something the fixer missed.

Tool calls made:
1. One grep for `trackedRel(`, `suspectBlobs(`, `writtenWorktreePaths(`, `failBatchInput{` and `DiskSizes(` across `internal`. My context lacked the full caller list for the F1 and F2 sets.
