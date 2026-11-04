```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for consistency with surrounding text and twins.

Finding 1 (LOW): the `Deps.CommitRecords` doc at `internal/sheddriver/drive.go:85` was left as it was. It says "commits the run's records at a done stop". `finishDone` now calls the seam a second time after the friction-failed hand-back's report is written.
- The doc does not say the seam may run twice, and it does not say it must tolerate a repeat call. Both batten and any later implementer should know this.
- The `finishDone` doc comment does say "committed again".
- Suggested fix: add a sentence to the seam doc.

No other staleness found:
- No other doc, `.md` file or comment names `holdsFile` or `dirHoldsFile`. The comment on `statusCommitPathspec` ("absent, unreadable, non-directory or file-less ... omits the entry") now matches `shedrun.HoldsFile` again.
- The `DriverRecordsRel` doc and the `paths.go` file header still hold.
- `sheddriver/doc.go:67` ("no `CommitRecords`: a done stop commits nothing") still holds, because `commitRecords` returns early when the seam is nil.
- The loomcli status pathspec and batten's `battenRunCommitPaths` both call `DriverRecordsRel`, so they stay in step.

Calls made (2, within the cap of 6):
- grep of `CommitRecords` and `holdsFile` in `.go` and `.md` files: my context did not show the `Deps.CommitRecords` doc or any `.md` references.
- The same grep, second pattern: the doc check for `holdsFile` and `dirHoldsFile` references.
