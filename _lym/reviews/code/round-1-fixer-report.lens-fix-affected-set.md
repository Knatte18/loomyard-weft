```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F1: the one every-seam-filled check (`internal/beamcli/dobbydeps_integration_test.go:75`) now lists `NewestAgentEnd`. No other seam-nil check exists, and `DobbyRunNotices` has only one production caller (`internal/beamcli/dobbydeps.go:165`), so the set is covered.
- F2: the only `RunNoticeDeps{` literal in a test is the harness in `internal/dobbyengine/runnotice_test.go:46`, and both new rows build on it. No other fake or constructor needs a nil-seam row.
- F3: `DobbyRunNotices` is called only from the two battencli tests, and the success-path test sits beside the absent-worktree test. Both cover the one seam.

Findings left unchecked: none.

Tool calls:
1. One `grep` over `internal` and `cmd` for `RunNoticeDeps{`, `DobbyRunNotices(`, `NewestAgentEnd` and the seam-nil checks. My context had no repo-wide caller or constructor search.
