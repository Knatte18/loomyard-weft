```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

F1 is the only finding. Its set is the sites that assert the shared `could not stop the recorded PR-Review session` prefix or carry the old row name.
- The new `wantErr` string is the only assertion of that prefix in any `.go` or `.md` file outside `_lym` and `.lym`.
- The old row name `PRReviewWithARecordedStrandStopsItThroughARunnerOfItsOwn` no longer appears anywhere, so no stale reference is left.
- The `probe reed before removing it` wording now in `wantErr` matches `internal/shuttleengine/stop.go:229`. The other hit, `internal/shuttleengine/wait.go:2123`, is a different log line and does not share the text the row asserts.
- The doc comment lines on the case were updated in step with the new row name.

Tool calls (1 of 6 used):
1. `grep -rn` over the repo for the prefix, the old row name and the `probe reed` wording. My context held only the two edited files and `stop.go`, not the rest of the repo.
