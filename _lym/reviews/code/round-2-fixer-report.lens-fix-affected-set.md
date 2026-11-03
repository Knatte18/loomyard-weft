```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun F1's set search (the removed exports, the dropped `time` import, and any other users) against the changed tree.

Findings: no findings.

Unchecked findings: none. The review has only F1, and its set is covered. No `DriftCaptureRequest` or `DriftCompareRequest` reference remains in any `.go` or `.md` file, and `driftwiring_test.go` has no `time.` use left, so dropping the `time` import is safe. The remaining test and export in those files (`TestCaptureSpawnRequest_StartsInTheAnchoredDirectory`, `CaptureSpawnRequest`) are untouched and still wired.

Tool calls:
1. `grep` for both removed export names and `time.` in the two edited files, plus a repo-wide grep for `loomcli.Drift(Capture|Compare)Request` in `.go` and `.md` files. My context did not hold the rest of the repo, so I could not tell whether other packages used the exports.
