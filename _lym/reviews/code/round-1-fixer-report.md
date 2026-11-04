# Fixer-report, code review round 1

All seven findings fixed as proposed unless marked.
Gate: `lym gate test` tier 0 over runproc, orchcli, beamcli, vscode and reedengine passed; tier 1 over beamcli passed.
`GOOS=windows go vet ./internal/vscode` is clean.

## Changes

- **F1** (commit e5dcb236): `proc.Reap(cmd)` after the successful `Start` in `spawnSession` (`internal/reedengine/lifecycle.go`) and in `Launch` of `launch_linux.go` and `launch_windows.go`; `launch_linux.go` now imports `internal/proc`.
- **F2** (commit 9cf130f3): `TestRunDobbyWith_TracesEachExitBeforeRun` counts the reap seam: once, before the wiring, after a placed worktree, and not at all after a refused placement.
  `orchCLI` gains a `reapInherited` field, set by `wireCLI` to the new `reapInheritedChildren` in `watch.go`; `watch` calls it, and `TestWatch` asserts it runs once in both rows. `newTestCLI` sets a no-op.
  Order against `LoadState` is not pinned: both rows reach the reap, and a move past the refusal return or the watcher start fails the count.
- **F3** (commit 758b2459, doc follow-up in the commit after it): `lister` gains `isAlive` beside `isRunning`.
  `liveness` replaces `live`: a gone pid or another start time removes the record; a zombie is left out of the result and its file kept.
  `TestRegisterAndList` asserts the file survives while the pid is a zombie and goes once it is gone.
  The `runproc` package doc says the same (self-check finding below).
- **F4** (commit 3836cb78): comments in `fabricengine/spawn.go` and `adoptcli/deps.go` say the child is reaped through `proc.Reap`.
- **F5**: `_lym/landing/summary.md` now names the `reap` and `reapInherited` seams, states that a zombie's record is kept and its end-sequence lister keeps it listed, drops the end-fake sentence, and lists the new tests.
  The file is under `_lym`, so it is edited and not committed.
- **F6** (commit 758b2459): `record_linux_test.go` `TestListAndEnderReadAZombieLeader` starts a real child, leaves it unreaped, registers it, and asserts `List` and `Owners` omit it while the end lister keeps it.
- **F7** (commit 758b2459): the field binding became `ender.recordLister()`, whose comment states why the running check is the alive check.
  F3, F6 and F7 touch the same files and share one commit.

## Gate repair

The verify run failed `TestHermeticGitEnv_GitSpawningPackagesHaveTestMain` in `cmd/lym`: `record_linux_test.go` held the token `exec.Command` and `internal/runproc` has no `HermeticGitEnv` test main.
The F6 test now starts its child with `os.StartProcess`, resolving `true` through `exec.LookPath`, so the file spawns no git-guarded command and the package needs no allowlist entry.
`lym gate test ./internal/runproc ./cmd/lym` passes at tier 0.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
- fix-consistency, MEDIUM: `internal/runproc/doc.go` still said `List` drops a zombie's record. Fixed as proposed.
- fix-consistency, LOW (three items: `proc/doc.go`, `PATTERN.md:79`, the `lister` comment): the fork itself found them consistent and proposed no change. Nothing to do.

## Disputed

None.

## Deferred

None.
