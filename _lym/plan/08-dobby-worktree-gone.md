# Card 8 — dobby-worktree-gone

**Edit:**
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#daemon`
- `internal/dobbyengine#daemon.loop`
- `internal/dobbyengine#daemon.recordStart`
- `internal/beamcli#hubDobbyDeps`
- `internal/dobbyengine#tidyShuttleFake`
- `internal/dobbyengine#tidyShuttleFake.StopStrand`

**Create:**
- `internal/dobbyengine#daemon.worktreeGone`
- `internal/dobbyengine#daemon.exitWorktreeGone`
- `internal/dobbyengine#TestRun_WorktreeGoneExits`
- `internal/beamcli#TestHubDobbyDeps_PrimeTidyShuttle`

**Extends:**
- `internal/dobbyengine/doc.go`
- `internal/beamcli/doc.go`

**Uses:**
- `internal/state#ReadJSONNoMkdir`
- `internal/logger#Episode`
- `internal/dobbyengine#newLoopHarness`
- `internal/dobbyengine#TidyShuttle`
- `internal/beamcli#runnerTidyShuttle`

**Intent:**
Decision 1's end of a beam dobby whose worktree a teardown removed: the root's absence cannot mark it, since every `internal/state` access recreates the record's directory, so the dobby tests the worktree's git link, which its own writes never recreate.

`worktreeGone` reports, for a beam dobby (`Geometry.Slug` non-empty) only, whether `filepath.Join(Geometry.WorktreeRoot, ".git")` does not exist; the prime's dobby never takes this exit.

`loop` calls it at the top of every iteration, before the probe tick's owner read and the tick, whatever woke the loop, a file event included; when it reports true, `exitWorktreeGone` runs and `loop` returns nil.
The loop tests it again after each tick, before `CheckBinary`, and exits the same way, so a worktree removed during a tick never meets a re-exec whose new image could not place the removed worktree and would leave the tidy in flight unstopped.

The kept tidy: `daemon` gains `keptTidy *TidyState`, which `recordStart` sets from the record it read.
After each tick the loop first reads the record through `state.ReadJSONNoMkdir` on `Geometry.StatePath` and its lock path, which creates no directory, reading an undecodable record as absent with one Warn as `ReadState` does, and then tests `worktreeGone` once; that test is the post-tick exit test above.
When `worktreeGone` reports true, the loop discards the read and its error and exits through `exitWorktreeGone` with the earlier `keptTidy`, since a tick that ends with the marker gone may have read a fresh empty `State` from a recreated record.
The rows that remove the worktree inside a tick reach this branch with a failed read, since the `Tick` seam is the tick's last step and the lock's directory is gone.
When it reports false and the read failed with an error other than a decode failure, the loop logs it through a `logger.Episode` of its own, a new `stateRead` field of `daemon` beside `ownerRead`, as `beamOwnedElsewhere` logs an unreadable owner, keeps the earlier `keptTidy` and goes on; a read that does not fail with such an error ends that episode with `Recover`.
When it reports false and the read succeeded, or found no record, or failed to decode, which reads as no record, `keptTidy` takes the record's `Tidy`, nil included, and nil for no record; the undecodable-record row below reaches the decode branch.

`exitWorktreeGone`:
- appends `<time> stop: the worktree of beam <slug> is gone` to `Geometry.PrimeLogPath` through `Deps.Log` with `rotate` false, since this process does not hold the prime's lock;
- when `keptTidy` is non-nil and `Deps.PrimeTidyShuttle` is set, stops its strand through `PrimeTidyShuttle.StopStrand` and appends `stopped the board-tidy agent <guid> of beam <slug>`, or on a failed stop `the board-tidy agent <guid> of beam <slug> did not stop: <error>; way forward: lym reed remove <guid>`, to the prime's `dobby.log` the same way;
- writes nothing at the removed worktree's path: it reads no state record, takes no state lock and creates no directory, and `Deps.Log` never creates one (card 6).

`Deps` gains `PrimeTidyShuttle TidyShuttle`, used only for that stop, whose runner is built over the prime's location, since the removed worktree's config is gone; nil stops nothing.
`hubDobbyDeps` sets it, for a beam dobby only, to a `runnerTidyShuttle` over the prime's location; finding no run record there, `StopStrand` removes the strand alone.

Bounds, stated in `doc.go`: a tick in flight when the worktree is removed can leave an empty `.lym/beam` stub holding the `state.json` and lock file its state access wrote, and the rest of that tick runs over the fresh empty `State` it read there; a tidy that tick spawned is not stopped, and one it settled is stopped again, which removes a strand only while reed still tracks it, and logs a failed stop when reed's status probe fails.

`tidyShuttleFake` gains `stopErr error`, which its `StopStrand` returns after recording the guid; its zero value returns nil, as every existing caller expects.

`TestRun_WorktreeGoneExits` (tier0) runs `newLoopHarness` with `Deps.Log` set to `dobbyengine.AppendLog`, so the rows read the files it writes, a `Slug`, a `WorktreeRoot` temp directory holding a `.git` file and the state record under it, a `PrimeLogPath` in another temp directory, a `tidyShuttleFake` as `PrimeTidyShuttle` and a `Watch` channel, with `stopAt` 3 in every beam row so the tick seam's cancel cannot end the loop before the exit a row pins; the tick hook removes the whole worktree on tick 1 and sends one watch event, with no probe due on the next iteration. Rows:
- a record holding a `Tidy` in flight: `Run` returns nil after one tick, the fake recorded one `StopStrand` of its guid, the prime log holds the exit line naming the slug and the stop line, and the worktree path does not exist after `Run` returns;
- the same with a fake whose `StopStrand` fails: the prime log's line names `lym reed remove <guid>`;
- no `Tidy`: the exit line alone and no `StopStrand`;
- a record with no `Tidy` at start that gains one through `UpdateState` during tick 1 with `.git` present, the worktree removed during tick 2: the fake recorded one `StopStrand` of that guid, the one the tick that ended with the marker present read;
- a record holding a `Tidy` at start that tick 1 clears with `.git` present, the worktree removed during tick 2: no `StopStrand`;
- a worktree removed during tick 1 with a `CheckBinary` fake that records its calls: `CheckBinary` is never called;
- a worktree removed after tick 1 has ended: in this row tick 1's hook sends no watch event, and a `Deps.Sleep` wrapper, on its first call only, removes the worktree, sends one value on the harness's `Watch` channel and returns, with no probe due: `Run` returns nil, the harness's recorded ticks hold exactly one entry (`len(h.ticks) == 1`), the worktree path does not exist after `Run` returns, the fake recorded one `StopStrand` of the kept `Tidy`'s guid, and the `CheckBinary` fake was called once, after tick 1, which checks only that nothing re-execs; a second tick, or a recreated `.lym/beam` stub at the removed path, fails the row;
- a record holding a `Tidy` at start whose `state.json` tick 1's hook replaces with a directory while `.git` stays present, sending no watch event, and the same `Deps.Sleep` wrapper removing the worktree on its first call: `Run` does not exit after tick 1, since the `CheckBinary` fake was called once, `keptTidy` stays the start record's, and the fake recorded one `StopStrand` of that `Tidy`'s guid;
- a record holding a `Tidy` at start whose `state.json` tick 1's hook overwrites with bytes that are not JSON while `.git` stays present, sending no watch event, and the same `Deps.Sleep` wrapper removing the worktree on its first call: the undecodable record reads as absent, `keptTidy` becomes nil, and the fake recorded no `StopStrand`;
- a prime dobby (empty `Slug`) whose `.git` is removed goes on ticking to `stopAt`.

`TestHubDobbyDeps_PrimeTidyShuttle` (tier1, internal to `beamcli`, a `hubforge` hub with one beam pair): `hubDobbyDeps` for the beam's geometry sets `PrimeTidyShuttle` to a `*runnerTidyShuttle` whose `worktree` is the prime's location, and for the prime's geometry leaves it nil.

**ImpactSummary:** One new exit at the top of the loop, active only for a beam dobby, plus one stop-only shuttle seam wired in `hubDobbyDeps`.

**Commit:** 8: End a beam dobby whose worktree is gone and stop its tidy agent
