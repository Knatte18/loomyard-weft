```yaml
lens: fix-proposal
outcome: reported
```

no findings

- F2 (`internal/beamcli/mergeparent.go:126`): the line now renders "Filed by `lym beam merge-parent X`." Nothing in the repo matches on the text "Filed by", so no consumer depends on the old spacing.
- F3 (`internal/boardengine/task.go:279`): the sentence matches the check at `task.go:306`, which accepts empty, on, off and `PullRequestMergeIn`. Task and project share that one check, so "for a task and a project alike" still holds.

Tool calls:
1. `grep` for "Filed by" across `.go` and `.md` files: to find consumers of the changed body text, which my context lacked.
2. `grep` for `PullRequestMergeIn` in `internal/boardengine`: to check the doc sentence against the validation code at `task.go:306`, which my context lacked.
