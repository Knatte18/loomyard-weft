```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the change set.

**Findings**

1. **LOW** — `internal/shuttleengine/codexengine/doc.go:156`. The package doc still says "The walk skips `.git` directories and follows no link". F5 changed the walk to skip the directories in `unreadScanSkippedDirectories` (`.git`, `node_modules`, `vendor`), and the doc comment of `unreadClaudeFiles` was updated to match. The package doc is the twin that was left behind, so a reader gets the old, narrower rule.

2. **LOW** — `internal/shuttleengine/codexengine/doc.go:154-155`. This passage says a CLAUDE.md beside the anchor's heddle-marked `AGENTS.override.md` "is not named". F4 widened the exemption to the no-`AGENTS.md` case with an imports-carrying CLAUDE.md. The sentence is still true, so this is only a cross-reference check, but "Heddle's render carries it in that file" is now also correct for the import case. No edit is needed beyond what finding 1 touches.

Other twins I checked agree: the `Prepare` doc comment and the `claudeFileIsUnread` logic; the `docs/overview.md` paragraph against `maxArgumentBytes` (128 KiB) and the item order in `Prepare`; the `verbs.go` help text against `renderEveryWorktree` (the prime, beams and task-pair listing and the failed-listing `render_warnings` case are both still true).

Tool calls:
1. grep for `.git` skip wording across the Go and markdown files: to find stale twins of the F5 change that my context lacked.
2. `sed -n 140,165p` on `internal/shuttleengine/codexengine/doc.go`: to read the package doc passage that grep surfaced.
