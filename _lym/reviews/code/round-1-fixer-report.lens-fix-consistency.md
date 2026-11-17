Task: check the fixer's change set for stale twins and sibling disagreements (fix-consistency lens).

**Findings**

- **LOW** — `internal/machinecli/start.go`, `startHub.Shortname` (`json:"name"`) and `startWorktree.Shortname` (`json:"shortname"`). In one envelope, the same hub shortname is now keyed `name` on a hub and `shortname` on each worktree. Only the hub's key was renamed, to give the readable summary a label. A consumer joining worktrees to hubs has to know both spellings. Evidence: `start.go:63` (`json:"shortname"`) against the hub field `Shortname string json:"name"`. Either rename the worktree key to match or note the difference in the field comment.

No other stale twins. I searched for each changed name and flag:

- `--no-prime-claim` and `NoPrimeClaim` appear consistently in `beamcli/cli.go`, `orchcli/start.go`, `orchcli/doc.go`, `machinecli/start.go`, `machinecli/doc.go` and the tests.
- `machinecli/doc.go`'s skip rule for a prime with no owner still matches the `planStart` skip text.
- `docs/overview.md`'s machine start paragraph (lines 867-872) still agrees with the change.
- No consumer of the old hub key `shortname` remains in Go code under `internal` or `cmd`.
- `writePrimeOwnerBoard` and the old `freedesktopDesktopDir` signature have no remaining callers.

**Calls made**
1. `grep` for `no-prime-claim` and `NoPrimeClaim` across the repo: to check every twin of the new flag and option. My context had no repo-wide view of them.
2. `grep` for `machine start` in `docs` and `README.md`: to find prose twins of the changed behaviour. My context held no docs.
3. `grep` for the old names (`freedesktopDesktopDir`, `writePrimeOwnerBoard`, `"shortname"`) in `internal` and `cmd`: to find callers of renamed things.
4. Read `docs/overview.md` lines 865-886: to confirm its machine start description still agrees.
