```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set, task: rerun F1's search for serial tests that capture the logger without a comment naming it, and check the fix covers every member.

F1 is the only finding with a set premise. No findings.

- **`spawn_test.go`:** the search lists three serial tests that use `logcapture`: `TestLoadOrInitStateLocked`, `TestLaunchStrandLocked_WriteFailureSendsTheFullLine` and `TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls`. All three now carry a comment naming the logger. The first already did, and the fixer added it to the other two.
- **Other files, out of scope:** the same search lists serial `logcapture` tests with no parallel or serial comment in nine other test files in `internal/reedengine`. The review scoped F1 to the tests this diff rewrote, and I did not check whether the diff touched these tests, so I do not count them as findings. The gaps are in `attach_test.go`, `launchscript_cleanup_test.go`, `lifecycle_test.go`, `namerepair_test.go`, `panebin_test.go` (two tests), `probe_test.go`, `reconcile_test.go` and `windowsize_test.go`. My awk was crude and may over-report. Check the list before acting on it.

Unchecked: none.

Tool calls:
1. Ran an awk loop over `internal/reedengine/*_test.go`, because my context did not hold the other serial `logcapture` tests, which F1's search needs.
