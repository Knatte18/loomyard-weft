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
- `plan:internal/dobbyengine#daemon.worktreeGone` -> `func (d *daemon) worktreeGone() bool`
- `plan:internal/dobbyengine#daemon.exitWorktreeGone` -> `func (d *daemon) exitWorktreeGone()`
- `plan:internal/dobbyengine#TestRun_WorktreeGoneExits` -> `func TestRun_WorktreeGoneExits(t *testing.T)`
- `plan:internal/beamcli#TestHubDobbyDeps_PrimeTidyShuttle` -> `func TestHubDobbyDeps_PrimeTidyShuttle(t *testing.T)`

**Extends:**
- `internal/dobbyengine/doc.go`
- `internal/beamcli/doc.go`

**Uses:**
- `internal/dobbyengine#newLoopHarness`
- `internal/dobbyengine#TidyShuttle`
- `internal/beamcli#runnerTidyShuttle`

**Intent:**
Decision 1's end of a beam dobby whose worktree a teardown removed: the root's absence cannot mark it, since every `internal/state` access recreates the record's directory, so the dobby tests the worktree's git link, which its own writes never recreate.

`worktreeGone` reports, for a beam dobby (`Geometry.Slug` non-empty) only, whether `filepath.Join(Geometry.WorktreeRoot, ".git")` does not exist; the prime's dobby never takes this exit.

`loop` calls it at the top of every iteration, before the probe tick's owner read and the tick, whatever woke the loop, a file event included; when it reports true, `exitWorktreeGone` runs and `loop` returns nil.
The loop tests it again after each tick, before `CheckBinary`, and exits the same way, so a worktree removed during a tick never meets a re-exec whose new image could not place the removed worktree and would leave the tidy in flight unstopped.

The kept tidy: `daemon` gains `keptTidy *TidyState`. `recordStart` sets it from the record it read; after each tick, while `worktreeGone` reports false, the loop reads the record and replaces `keptTidy` with its `Tidy`, nil included. A tick that ends with the marker gone keeps the earlier value, since it may have read a fresh empty `State` from a recreated record.

`exitWorktreeGone`:
- appends `<time> stop: the worktree of beam <slug> is gone` to `Geometry.PrimeLogPath` through `Deps.Log` with `rotate` false, since this process does not hold the prime's lock;
- when `keptTidy` is non-nil and `Deps.PrimeTidyShuttle` is set, stops its strand through `PrimeTidyShuttle.StopStrand` and appends `stopped the board-tidy agent <guid> of beam <slug>`, or on a failed stop `the board-tidy agent <guid> of beam <slug> did not stop: <error>; way forward: lym reed remove <guid>`, to the prime's `dobby.log` the same way;
- writes nothing at the removed worktree's path: it reads no state record, takes no state lock and creates no directory, and `Deps.Log` never creates one (card 6).

`Deps` gains `PrimeTidyShuttle TidyShuttle`, used only for that stop, whose runner is built over the prime's location, since the removed worktree's config is gone; nil stops nothing.
`hubDobbyDeps` sets it, for a beam dobby only, to a `runnerTidyShuttle` over the prime's location; finding no run record there, `StopStrand` removes the strand alone.

Bounds, stated in `doc.go`: a tick in flight when the worktree is removed can leave an empty `.lym/beam` stub holding the `state.json` and lock file its state access wrote, and the rest of that tick runs over the fresh empty `State` it read there; a tidy that tick spawned is not stopped, and one it settled is stopped again, which removes a strand only while reed still tracks it, and logs a failed stop when reed's status probe fails.

`tidyShuttleFake` gains `stopErr error`, which its `StopStrand` returns after recording the guid; its zero value returns nil, as every existing caller expects.

`TestRun_WorktreeGoneExits` (tier0) runs `newLoopHarness` with `Deps.Log` set to `dobbyengine.AppendLog`, so the rows read the files it writes, a `Slug`, a `WorktreeRoot` temp directory holding a `.git` file and the state record under it, a `PrimeLogPath` in another temp directory, a `tidyShuttleFake` as `PrimeTidyShuttle` and a `Watch` channel; the tick hook removes the whole worktree on tick 1 and sends one watch event, with no probe due on the next iteration. Rows:
- a record holding a `Tidy` in flight: `Run` returns nil after one tick, the fake recorded one `StopStrand` of its guid, the prime log holds the exit line naming the slug and the stop line, and the worktree path does not exist after `Run` returns;
- the same with a fake whose `StopStrand` fails: the prime log's line names `lym reed remove <guid>`;
- no `Tidy`: the exit line alone and no `StopStrand`;
- a record with no `Tidy` at start that gains one through `UpdateState` during tick 1 with `.git` present, the worktree removed during tick 2: the fake recorded one `StopStrand` of that guid, the one the tick that ended with the marker present read;
- a record holding a `Tidy` at start that tick 1 clears with `.git` present, the worktree removed during tick 2: no `StopStrand`;
- a worktree removed during tick 1 with a `CheckBinary` fake that records its calls: `CheckBinary` is never called;
- a prime dobby (empty `Slug`) whose `.git` is removed goes on ticking to `stopAt`.

`TestHubDobbyDeps_PrimeTidyShuttle` (tier1, internal to `beamcli`, a `hubforge` hub with one beam pair): `hubDobbyDeps` for the beam's geometry sets `PrimeTidyShuttle` to a `*runnerTidyShuttle` whose `worktree` is the prime's location, and for the prime's geometry leaves it nil.

**ImpactSummary:** One new exit at the top of the loop, active only for a beam dobby, plus one stop-only shuttle seam wired in `hubDobbyDeps`.

**Commit:** 8: End a beam dobby whose worktree is gone and stop its tidy agent
