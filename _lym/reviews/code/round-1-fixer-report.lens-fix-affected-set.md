```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I reran each set-premised finding's search once and found nothing blocking.

**Findings**

- LOW, F12 / `internal/shedadapters/burlerreview.go` lines 392, 424, 573, 656 and 675 (`archiveRound()`): the review producer also retires a round's review and acceptance, but it never clears `fixer_rerun`.
  - Only `BurlerFixProducer.archiveWholeRound` clears it (`burlerfix.go:128-130`).
  - Evidence: the grep shows `archiveRound()` called five times in `burlerreview.go` and none of them reaching `clearRerun`.
  - The digest key still guards a changed review, so the leak is the same narrow case F12 described: an identical-bytes re-accepted review inherits the old halt. The review named only the fix producer's function, so this is a sibling left as it was.
- LOW, F6 / `burlerfix.go:313` (the pre-attempt `recordRerun`) and `burlerfix.go:292`: an attempt 2 that archives nothing now replaces `rerun.Reason` in memory but records nothing.
  - If the call then exits through a path that records nothing (`ErrHalfNotStopped` via `errorExit`), the stored record keeps the older reason.
  - That matches the pre-attempt-archive-only design the review asked for, so it is a gap in the set rather than a defect.

**Sets that held**

- F1: every `remotes` read and write goes through `configRoot` or the `key` derived from it (`cluster.go:300`, `359-381`, `387-397`).
  - `recordClone` (`cluster.go:334`) passes the fresh clone root, which `configRoot` would leave unchanged, so it needs no change.
- F2: the attached `--config-env=` form is handled (`cluster.go:571`), and the separate-word form and `-c` are unchanged.
- F3 and F5: `leadsInto` is called only from `pushMutates` and `recordRemote`; clone uses `sourceLeadsIn`.
- F4, F11: `forkViolation` and `mutatingGitCommands` have no callers outside `cluster.go` and its test.

**Calls made**

1. One `grep` over `cluster.go` for the `remotes`, `setRemote`, `record`, `rootOf`, `leadsInto` and option-handling sites, plus `shedadapters` callers of `archiveReport`, `recordRerun`, `archiveWholeRound`, `clearRerun` and `archiveRound`, plus callers of `mutatingGitCommands` and `forkViolation` elsewhere in `internal`. My context held the old file text, not the post-edit call sites or the `burlerreview.go` callers.
