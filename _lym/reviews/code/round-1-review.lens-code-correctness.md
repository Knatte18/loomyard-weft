```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the `internal/runpusher` diff. I walked each changed function and branch with a concrete input.

**List**

- `worktreeGone(path)`, `targetreads.go:86`:
  - **Standing worktree** (`.git` is a file or a directory): `Lstat` succeeds and it returns false, nil. Correct.
  - **Path gone:** `Lstat` gives ENOENT, which matches `fs.ErrNotExist`, so it returns true, nil. Correct.
  - **Directory left without `.git`:** ENOENT, true, nil. Correct.
  - **`.git` is a dangling symlink:** `Lstat` succeeds and the worktree reads as standing. That is acceptable, since a removal leaves no such link.
  - **Path is a regular file, not a directory:** `Lstat` of `<file>/.git` returns ENOTDIR, which does not match `fs.ErrNotExist`, so it returns an error. The old `os.Stat(path)` returned "not gone" here. This is a behaviour change (finding 1).
  - **Empty path:** this would stat `./.git` relative to the working directory. No caller reaches it: `PruneLedger` skips `entry.Worktree == ""`, and a Location always has a non-empty path. Correct.
  - **Permission denied:** the error is wrapped and returned. Correct.
- `TargetOf`, `target.go:75`:
  - **Gone pair path:** it returns `{l, slug, Pair: true}`, nil, with no git read. Correct.
  - **Prime path:** `.git` stands, so `IsPrimeWorktree` runs, which is unchanged. Correct.
  - **Stat error:** it returns the pair target together with the error, not `Target{}` (finding 2). Every caller I read (`commitstatus.go:170,204`, `wiring.go:216,1413`, `pusher.go:78`) discards the target on error, so no wrong behaviour results.
- `starter.start`, `start.go:115`, with a gone pair:
  - **Start and StartIfAhead:** both return false, nil before any ledger, ahead, lock or refusal read. Nothing is written. Correct.
  - **Gone read fails:** false and the error are returned. Correct.
  - **Non-pair targets:** the gone read is never called. Correct.
- `TargetRefused`, `targetreads.go:100`:
  - **Gone pair:** returns false, nil.
  - **Consumers:** `settleIfRefused` in `dobbydeps.go` now sends the target on to `RestartPusher`, which ends silently at the start's gone check. `refusedPushWayForward` already returned not-refused for a missing directory, and the leftover-directory case now reads the same way.
  - **`landingdeps.go:1042`:** not reached for a gone parent. `parentPairLocation` (via `ResolveWorktree`) and `ParentTarget` (via `IsPrimeWorktree`) fail first, at `landingdeps.go:857` and `parentseams.go:30`.
  - **Gone beam pair whose ledger entry holds landings:** `parentTargets` gives a pair target, the refusal read answers false and the start skips silently. The earlier warning is now absent (finding 3).
- `NewDeps` `WorktreeGone`, `run.go:98`:
  - **Prime, board and landing targets:** the prime's `.git` directory stands, so they read not gone. Correct.
  - **Pair removed mid-run:** `canPush` returns false and the pusher exits, as before.
  - **Leftover directory:** the pusher now also exits, where before it went on to git reads that failed. Correct.
- `PruneLedger`, `targetreads.go:141`:
  - **Leftover-directory entry with no landings:** dropped through `pushledger.Update`, which re-checks landings under the lock. Correct.
  - **Stat error:** the entry is kept and the error joined. Correct.
  - **Entry with only `HandOvers` or `Dropped` on a leftover directory:** now dropped, where it was kept before. A missing path already got this treatment, so this matches the existing semantics.

**Findings**

1. **LOW**, design, `internal/runpusher/targetreads.go:86-95`.
   - When the worktree path is a regular file rather than a directory, `Lstat(path/.git)` returns ENOTDIR. `worktreeGone` reports that as an error, not as "not gone" (as before) or "gone".
   - `TargetOf`, `TargetRefused`, the start's gone read and `PruneLedger` then all fail on it every time. `PruneLedger` keeps such an entry for good, and each tick logs a prune-failure episode.
   - Evidence: `targetreads.go:87-94`, and the old `os.Stat(path)` in the diff at `run.go:-49`.
2. **NIT**, consistency, `internal/runpusher/target.go:77-79`.
   - On a failed worktree read, `TargetOf` returns a filled pair target together with the error. Its other failure path, at `target.go:83`, returns `Target{}`.
   - No caller uses the value: `commitstatus.go:170-174` and `wiring.go:216-220` return on error.
3. **LOW**, design, `internal/runpusher/start.go:119-124` and `targetreads.go:101-106`.
   - Take a beam pair whose ledger entry still holds landings and whose worktree was removed outside `internal/pairteardown`, which refuses while a landing entry stands.
   - `parentTargets` (`dobbydeps.go:204-222`) lists it, `TargetRefused` reads false and the start returns false, nil. `PruneLedger` keeps the entry because it holds landings (`targetreads.go:148`).
   - The owed landing therefore stays owed with no warning from the start path. Before the change, each episode logged the start's failure.
   - Only the push summary's owed-age notice still surfaces it, if `FirstOwed` is set.

One tool call: I read `loomcli/landingdeps.go:849` `parentPairLocation` and `runpusher/parentseams.go:29` `ParentTarget`, to see whether `parentPushState` reaches `TargetRefused` for a gone parent. The manifest held neither definition.
