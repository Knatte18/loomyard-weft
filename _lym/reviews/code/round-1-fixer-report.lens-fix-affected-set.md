```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Set-premised findings I checked:
- **F3** (stale "only empty, on, off accepted" wording): I searched `internal` for other places that list the accepted `pull_request` values. Every site that covers the entry field already names `merge-in`:
  - `boardengine/task.go:45`
  - `boardengine/task.go:307`
  - `boardengine/doc.go:230`
  - `boardcli/cli.go:549`
  - `landingshed/pullrequestmode.go:72`
  The remaining on/off hits are about the parent's beam.yaml mode (`on`, `off`, `required`, `merge`), where `merge-in` doesn't belong.
- **F2** (missing space before the backtick): `Filed by` occurs only once, at `beamcli/mergeparent.go:126`, and the space is now there.

Findings I left unchecked:
- **F1** is a decision-class finding that went to the parent, so there was nothing for me to check.
- **F4** is the fixer's dispute, not a change to this change set.

Tool calls (1):
1. A grep over `internal` for the accepted-value wording and for `Filed by`, which covers the F3 and F2 searches in one call. My context held only the files the fixer read, not the rest of the tree.
