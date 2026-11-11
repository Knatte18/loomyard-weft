```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Checked and found consistent:
- **`failedpush.go` pairSlug doc:** the added sentence about a records branch with a paired code worktree matches the code. `pairedCodeWorktree` fills `CodeWorktree` for every weft sibling, and `Prime` is set only for the code side.
- **Other docs on the failed-push run tag:** `fabricengine/doc.go`, `orchcli/doc.go`, `pairteardown/doc.go` and `creelengine/doc.go` say nothing that contradicts it.
- **Run-mail drop doc in `pairteardown/doc.go:116`:** still agrees with the `PATTERN-pair-remote.md` bullet.
- **`PATTERN-pair-remote.md` reorder:** the board-settle bullet is whole again. The "running record" lines continue "releases the claim", and the run-mail bullet follows as its own bullet.
- **`teardown.go` doc order:** the `RemovePair` self-reference now sits ahead of the run-mail paragraph.
- **Renamed tag helpers:** `runNoticeTags` and `runNotifyTags` have no remaining inline duplicates. The old inline `PostTags` literals in `beamcli/dobbydeps.go` and `battencli/notices.go` were replaced.
- **`startCreelDaemon` busy receiver:** the `delivery.SetIdle(receiver, false)` change touches the orchcli helper only. beamcli has its own separate `startCreelDaemon`.

Tool calls made:
1. One grep over the package docs and `pattern/` for run-tag and records-branch statements, to find twin text that the doc changes might contradict. My context held only the edited files, not these docs. The same call listed orchcli `runmail*_test.go` files; none exist.
