```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I checked F1, F2, F7 and F8 for set members the fix missed. F7 left one stale member; F1, F2 and F8 are covered.

1. **LOW**, F7 prune prose. The comment on `Prune` in `internal/boardengine/store.go:781` still reads "Prune removes every done entry, strips the removed slugs from the survivors' depends_on and order…". The code at `store.go:787` is `if isDone(t) && !t.NotifyOnLand`, so a flagged done entry survives. The sibling comment at `internal/boardengine/board.go:404` already says "bar one still carrying notify_on_land". The comment on `PruneBoard` in `internal/battenshed/land.go:72`, "prunes the done tasks off the board", is the same claim in softer form. The F7 fix corrected the CLI help, `docs/overview.md`, `contracts/orchrole/orch-role.md` and `plugins/ly/skills/board/SKILL.md`, but not these Go doc comments. Neither file is in `git status`, so this round did not touch them.

No findings for the other sets:
- **F1:** the unread-status sites in `deliver.go`, `flags.go`, `pool.go`, `verbs.go` and `fold.go` read or deliver mail and never return it. The return path runs only through `sweepSubjects` and `releaseGoneHolder`, and both carry the `waitsForHubOrch` exclusion.
- **F2:** the `NotifyOnLand` consumers in `view.go` and `overviewdobby.go` read it for display only. The only place a flagged done entry is taken up is `takeDoneEntries`, which now skips `OwnedElsewhere`.
- **F8:** `landedNotice` has a single caller, the single-entry upsert at `internal/boardcli/cli.go:558`. Upsert-batch does not call it.

Calls made, none beyond the four greps:
1. A grep for prose about prune and done entries, since my context held only the prune sites the review named.
2. A grep for `StatusUnread` in `internal/creelengine` non-test files, to list every unread-mail site and not only `sweep.go`.
3. A grep for `landedNotice` and `NotifyOnLand` in non-test Go files, to list every consumer of the flag and every caller of the notice.
4. A grep for `Prune` callers and `notify_on_land` in markdown files, to check the prune callers and any other doc that mentions the flag.
