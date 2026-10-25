```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind F1–F4 and check the fix covers every member.

No findings.

Findings I checked:
- **F1 (`RunCompare` callers):** There are two production callers, `internal/driftcli/cli.go:232` and `internal/loomcli/driftwiring.go:204`, and both now set `LockRetry`. Every `compare_test.go` request comes from `newCompareFixture`, which sets `LockRetry: time.Second`.
- **F2 (ruling wording):** No other doc comment or doc text describes rulings as a partial-scope behavior. The only remaining "partial scope" hits concern the skip list, which is correct.
- **F3 (cancel and `killCapture`):** The only production call sites are `EndSession` (`teardown.go:265`) and the new one in `EndForCancel` (`cancel.go:102`). `killPairCapture` is wired once, at `teardown.go:193`. The `loop_windows.go` use of `TerminateNamedJob` is the loop job, not the capture job.
- **F4 (`CaptureRequest.Head` and `Check`):** `RunCapture` is called by `driftcli` `runCaptureChild` and by `loomcli` `driftwiring` (`Check: checkMain`); both now return the HEAD they verified. `RunSideCapture` (`cli.go:294`) builds a `CaptureRequest` but does not use `Head` or `Check`, and the build passes, so removing `Head` broke nothing.

Findings I left unchecked: none.

Tool calls:
1. One combined `grep` across `internal`, `docs` and `PATTERN.md` for the `RunCompare`/`CompareRequest`, `RunCapture`/`Check`/`Head`, `killCapture`/`TerminateNamedJob` and "partial scope" sets. My context held the files I edited but not the full caller sets.
