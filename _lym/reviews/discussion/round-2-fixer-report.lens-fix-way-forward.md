```yaml
lens: fix-way-forward
outcome: reported
```

fix-way-forward lens: 2 findings.

**MEDIUM: validate-review's shape-checks fallback is unreachable outside a worktree.**
- **Text:** D3, "`lym burler validate-review` resolves its worktree as the other burler verbs do, … A file outside a run directory gets the shape checks alone."
- **Problem:** if the verb resolves its worktree the way the other burler verbs do, it aborts before it reaches the stated fallback whenever the file is not inside a git worktree.
- **Evidence:** the other verbs' pre-run calls `clihelp.Abort(ctx, 1)` when resolution fails (`internal/burlercli/cli.go:75-80`). Today the verb skips that pre-run (`cli.go:68`), and its help says it "needs no hub or git repository" (`internal/burlercli/validate.go:36`).
- **Way forward:** resolve the worktree only when the file sits in a run directory, and fall back to the shape checks when resolution fails.

**LOW: the parent is never told that only an added entry counts as its answer.**
- **Text:** D2, "A parent's answer reaches round N only as an `Added after Discussion` entry … written through `lym loom decision add`", together with D3's rule that `answer` must quote the entry's title.
- **Problem:** D2 and D3 now accept a parent's answer through this one channel only, but the decision escalation the parent reads names only `continue` and `accept`. A parent who answers in any other way leaves the finding `open` and carried, so it is escalated again with no progress. No changed text or acceptance criterion tells the decision fragment to name `lym loom decision add`, one entry per answered finding.
- **Evidence:** `decideVerbs` offers only the `lym loom circling continue`/`accept` lines (`internal/shedadapters/bouncerdecision.go:279-289`).

Tool calls:
- grep of `internal/burlercli` for the validate-review verb's resolution path: my context held only `cli.go:60-70` and not the verb's own file, `validate.go`.
