```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- Twin check: `spawn_test.go` has three tests that call `logcapture`: `TestLoadOrInitStateLocked`, `TestLaunchStrandLocked_WriteFailureSendsTheFullLine` and `TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls`. The first already had its serial comment (line 139) before this change. The other two have one now (lines 544 and 644), so no uncommented serial test is left in the file.
- Placement: each new line sits above the `//testtiming:keep` line, as the review asked.
- Wording: both lines read "It captures the process-global logger, so it does not run in parallel."
- That matches the package's existing serial-test comments, for example `windoworder_test.go:80` and `ensuresession_integration_test.go:151`.
- Scope: the commit changes only `spawn_test.go`, with two added lines.

Tool calls (1 of 6):
1. `grep` over `spawn_test.go` for `logcapture` uses with their serial comments, plus `git show --stat HEAD`. My context did not hold the comments of the other tests in that file or the commit's scope.
