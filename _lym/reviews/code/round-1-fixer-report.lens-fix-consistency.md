```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for stale twins and cross-references (lens fix-consistency).

no findings

- F2: the other docs that name `interrupted-stop` (`internal/loomcli/doc.go:6`, `internal/loomcli/looparm.go:182`) don't restrict the state, so they still agree with `case interrupted:`. The only caller, `internal/loomcli/halt.go:65`, passes `n.State` through unchanged, so a `running` or `done` interrupted note now gets the anomaly as intended. The three doc comments in `internal/loomengine/anomaly.go` agree with each other and with the code.
- F1: the twin `loopExitStop` (`internal/shedverbs/loopwait.go:474`) already falls back with `cmp.Or(before.CurrentProducer, rec.CurrentProducer)`, so both paths now use the same rule.
- F3: `internal/gitexec/doc.go:49-52` states 250 ms doubling for local and remote subcommands. The new rows match it, and `SetStartFailuresForTest` had no other callers to update.
- F4–F8: these changes touch only tests, and no sibling doc or row went stale.

Tool calls made:
1. `grep` across `.go` and `.md` files for "failed or paused", "interrupted-stop" and similar phrasings, plus `internal/gitexec/doc.go`. Context held no doc or sibling text outside the touched files.
2. `sed -n 40,90p internal/loomcli/halt.go` plus `git diff --stat`. Context lacked `ClassifyHalt`'s caller and the full list of changed files.
