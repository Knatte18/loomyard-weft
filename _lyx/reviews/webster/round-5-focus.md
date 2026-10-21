---
round: 5
focus:
  - "internal/gitrepo/gogit.go `reindexIfPacksChanged` and `readGoGit`: settle whether any path lets a reader holding only `r.goGitMu.RLock()` trigger or observe go-git's lazy pack-index rebuild (`requireIndex`) after a `storer.Reindex()`, and whether the rebuild is now forced while the exclusive lock is held."
  - "PATTERN.md overview against main's text (`git diff main -- PATTERN.md`): settle whether any entry other than `PATTERN-origin-reconcile` and `PATTERN-push-failure-visible` differs from main's overview line, and whether the two declared entries carry their declared text."
  - "internal/reedcli/watchdog_integration_test.go shared watchdog loop: settle whether `timing.IdleCycles` spans the `Up` call plus the longest wait `DownThenUpDoesNotKillDaemon` uses, and whether any later subtest relies on an idle exit."
---

The focus names the three places the last round's findings touched, so a fix there gets a fresh read.
