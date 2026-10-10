# Webster round 3 — fixer report

Both findings were validated and fixed, one commit each.

## Fixed

- **F1** (`15e310366`) — the publish failure is real, but its cause is reed's pane reap, not the kit's sweep.
  The leaked pid 3593964 falls between server pids 3592600 and 3594223 on socket `lyx-warp-bare-LYXHUB-1cbd6fef`, which are the two incarnations in `TestSmokeStateRecovery/StaleStateFileIsNotMistakenForLiveStrands`.
  That step adds the `generation-1` strand (`sleep 300`) and runs `down` at once.
  `Down` snapshots the pane descendant closure before `kill-session`, so a `sleep` the pane shell forks after the snapshot is outside it, and one not yet moved into the terminal's foreground misses the hangup.
  The server was killed by reed's `down`, not by `sweep`, so the review's step 1 would not have caught it.
  The fix: `reapPaneChildren` in `internal/reedengine/lifecycle.go`, shared by every pane-destroying path, reaps the snapshot as before, then reaps every other process still in a session a snapshotted pid led, within the same deadline.
  The session id comes from `/proc/<pid>/stat` through `parseStat`, which replaces `parseStatPPID`, and from a new `sessionMemberPIDs` probe (Linux reads `/proc`; Windows returns nil, having no POSIX sessions).
  `TestParseStat` gains the session column and a non-numeric-session row.
  The test change is a coverage gap, so it extends the existing step `TestSmokeTeardown/DownForceKillsSighupImmunePaneChildren` rather than adding a test.
  Its immune payload now forks a late immune child five seconds in, after `down`'s snapshot, and the step asserts that child is gone when `down` returns.
  The step also now waits for the payload to start before snapshotting.
  Without that wait, `down` ran in 0.3 s, before the pane shell had started the payload, so the step had been passing with nothing immune to reap.
  With the session pass disabled the step fails, and the kit prints the exact publish-failure line `left running ... ["sleep" "300"]`; with it, the step passes in 15.5 s.
  The review's step 2 is done: the kit's leftover line now names the process's cwd (`TestAfterRun_LeftoverProcesses` asserts the full line), and `PATTERN-test-isolation.md` says so.
  Steps 3 and 4 were not done: the sweep is not where the process escaped, and the reap-descendant chain's `sleep` is now covered by the session pass.
- **F2** (`bc0864a4b`) — `gitWorktreeRoot` resolves the root with `filepath.EvalSymlinks`, as `git rev-parse --show-toplevel` did; a resolve failure maps to the bare `ErrNotAGitRepo` like every other failure there.
  `TestResolve_AnchorScenario` gains a "symlinked start" step, linked through `fslink.CreateDirLink` and skipped on Windows, where the link is a junction that `EvalSymlinks` leaves unresolved.
  It fails without the fix (`WorktreeName:link`, a second `HubPath`) and passes with it.
  `dotgit.FindRoot`'s doc now states that it returns the walked root with symlinks unresolved, and `docs/shared-libs/lyxcwd.md`'s `Resolve` paragraph states the resolution.

Verified with `lyx gate test ./...`, `lyx gate test --tags integration ./...` and `lyx gate test --tags tmux ./...` (the publish tier), all green, plus `lyx loom lint-comments` and `GOOS=windows go vet ./internal/reedengine`.

One slip to report: while polling a background run I invoked `lyx burler await-review` with `/dev/null` instead of this round's marker path.
It is read-only and changed nothing, but it breaks the rule that the wait verb takes only the marker path.

## Disputed

None.

## Deferred

None.
