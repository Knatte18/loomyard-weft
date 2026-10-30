```yaml
lens: fix-affected-set
outcome: reported
```

Fix-affected-set lens: no findings.

- **F1 (closure wiring):** `TaskHasCommits` is built in one place, `landingdeps.go:337`. The new integration test covers it on three paths: a level branch, one commit ahead, and an unresolvable parent branch. The other fixtures set the seam by hand: `envkit.go:210`, `prreview_test.go` and `finalize_pr_test.go`. `wiring_seams_test.go:44` and `battencli/wire_seams_test.go:101` list it as intentionally unset.
- **F2 (error rows):** `TaskHasCommits` is read in exactly two places, `prreview.go:84` and `finalize_pr.go:121`. Both now have an error row.
- **F3 (rejected token in Finalize):** every GitHub call site in `finalize_pr.go` goes through `fz.githubFailure`, so the one new rejected-token row covers the ordering for all of them.
- **F4 (`CommitLog` read for a boolean):** the other `CommitLog` callers are `landingdeps.go:334`, which needs the subject lines, and `peers.go:289`, which is outside the finding. The `websterengine` callers use a different `CommitLog` and are out of scope. No other caller uses `CommitLog` only for `len(...) > 0`.

Nothing is left unchecked.

Tool calls:
1. One grep for `CommitLog`, `HeadHoldsCommitsNotIn` and `len(commits) > 0` across `internal`, to check F4's set of callers. My context didn't hold the callers outside `loomcli`.
