```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Sets checked:
- **F1 (listing sites).** Other `fabricengine.List` callers filter staging in a different way or not at all: `orchcli/live.go` skips staging and prunable entries, while the rest are outside the heddle render fan-out. `namedWorktree` (`internal/heddlecli/cli.go:116`) still matches a staging entry by leaf name and would accept `lym heddle render <name>` for it. That is the hand-render path, not the fan-out the review flagged. The review did not ask for it, so I mark it out of scope.
- **F5 (docs).** The `--no-render` and "live worktree" wording in `cli.go:58-61` and `pattern/PATTERN-heddle-instructions.md:24` does not name `rendered` or `failed`. Neither needs the `render_warnings` addition.
- **F2 (`--no-render` sites).** The flag reaches `finishWrite` at four sites in `verbs.go`: `set` of a repository entry, `setOverride`, and `remove` with and without `--layer lym`. The new test steps drive three of them: `set --layer lym --no-render`, `remove --layer lym --no-render` and `remove --no-render`. `set` of a repository entry already had its `--no-render` step before the fix.
- **F4 (way-forward sites).** The `err` and `Unwritten` cases both go through `withRenderWayForward`. `default` carries no failure, so it needs none.

Calls made:
1. `grep` for `fabricengine.List(` across `internal` (non-test) and for `render_warnings|live worktree|no-render|prunable` across `pattern`, `docs` and heddle sources, to find the other listing sites and doc mentions.
2. `Read` of `internal/heddlecli/cli.go:52-141`, because my context lacked the group help text and `namedWorktree`.
