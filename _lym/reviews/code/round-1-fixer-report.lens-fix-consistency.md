```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F2 and F3 edits for twins or siblings left stale.

No findings.

- The F3 doc sentence agrees with the refusal at `internal/boardengine/task.go:307`, which accepts on, off, `PullRequestMergeIn` and empty.
- My grep over `internal/boardengine` and `internal/beamcli` found no other doc sentence that lists only empty, on and off.
- The F2 body line at `internal/beamcli/mergeparent.go:126` now matches the pre-diff text.

Tool calls:
1. Bash grep over `internal/boardengine` and `internal/beamcli` for other lists of accepted `pull_request` values and for the "Filed by" twin. My context held neither the other doc sentences nor the other files.
