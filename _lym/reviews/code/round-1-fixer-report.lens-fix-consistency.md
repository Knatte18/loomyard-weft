```yaml
lens: fix-consistency
outcome: reported
```

No findings.

I checked the sibling text that says "live worktree" against the new skip rules (prunable, in staging, not a direct child of the hub). It still agrees.
- `internal/heddlecli/cli.go:60` and `pattern/PATTERN-heddle-instructions.md:24` use the same wording and don't enumerate exclusions, so neither needs a change.
- The `doc.go` and `verbs.go` doc comments, and the `set` and `remove` help text, now name `render_warnings`, the staging skip and the out-of-hub skip consistently.
- The `render` verb keeps its own way-forward wording. That is not a twin of `withRenderWayForward`, because it deliberately omits the worktree name.

Calls made:
1. `grep` on `pattern/PATTERN-heddle-instructions.md` and the repo for "live worktree" wording, because my context lacked the pattern file and other sibling mentions.
2. `sed` on `internal/heddlecli/cli.go:50-75`, because my context lacked that file's group help text.
