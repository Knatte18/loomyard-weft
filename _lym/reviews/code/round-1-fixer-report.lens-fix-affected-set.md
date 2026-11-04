```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches for F2/F3 (holdsFile copies and callers), F4 (CommitRecords sites) and F1.

No findings. The fix covers every member of each set I could check.

- **F3, `holdsFile` callers:** none remain in `internal/loomcli`. The three `wiring.go` callers and the one in `landingdeps.go` now call `shedrun.HoldsFile`. The only other first-file walk is `driftengine/capture.go`. It returns `(bool, error)` for a different purpose, so it is not a duplicate of this helper.
- **F2, `DriverRecordsRel`:** its one callers' path, `dirHoldsFile`, is now `HoldsFile`, which carries the `IsDir` guard. Both drivers, loom's `statusCommitPathspec` and batten's `battenRunCommitPaths`, get the guard through `DriverRecordsRel`.
- **F4, done paths:** `finishDone` is called from one place (`sheddriver/drive.go:254`). Both `CommitRecords` wirings (`loomcli/drivewiring.go:258` and `battencli/drive.go:327`) re-run the same pathspec, which includes the drive-reports directory. The second call is a no-op when nothing changed, so the repeat is harmless.
- **LOW, stale wording:** the `CommitRecords` field comment says "commits the run's records at a done stop" (`sheddriver/drive.go:85`). It is now also called after a friction-failed done's report is written. The comment is still true, so I leave it to the fixer whether to mention the second call.
- **Checked and left alone:** `sheddriver/doc.go:67` says "no `CommitRecords`: a done stop commits nothing". That stays true when the seam is nil, because `commitRecords` returns early.
- **Unchecked:** F1 is a coverage gap on one seam, not a set, so I did not search it.

Tool calls:
1. Bash grep for `holdsFile(` in `internal/loomcli`, files using `fs.SkipAll`, and `finishDone`/`CommitRecords`/`DoneFrictionFailed` sites across `internal`. My context had no repo-wide caller search.
2. Bash `sed` of `loomcli/drivewiring.go` lines 245-275 and `sheddriver/drive.go` lines 80-88. My context lacked loom's `CommitRecords` wiring and the field comment.
