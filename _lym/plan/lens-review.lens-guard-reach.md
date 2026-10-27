```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: whether each guard, refusal and test row the plan adds can be reached from a real entry point, and whether a guarded site recurs unguarded.

**BLOCKING — Card 10, Decision 3 (`placeDobby` refusal after resolve): the re-exec test recreates the `.lym/beam` directory the guard depends on.**
- Step 1 says a worktree with no `.lym/beam` directory gets no line and no directory, and that it then runs `dobbyengine.ReadsAsReexec` on that directory's `state.json`.
- Card 7 builds `ReadsAsReexec` on reading the record, so it goes through `ReadState`, which calls `state.ReadJSON`.
- `state.ReadJSON` calls `os.MkdirAll` on the record's directory before it takes the read lock (`internal/state/state.go:93`, `:96`).
- So every refusal on a resolved worktree that was never a beam creates `<worktree>/.lym/beam` and its `state.json.lock`.
- Card 10's own row ("the task pair without a `.lym/beam` directory gets no line, no notice and no directory") fails as planned.
- The "no directory" guard state cannot hold at that point.
- `internal/state/state.go:175` has `ReadJSONNoMkdir`; neither card names it, or a stat of the directory first.
- The same recreation happens in step 3: the wiring failure runs `ReadsAsReexec` on `geom.StatePath`. That one is benign only when the directory already exists.

**MEDIUM — Card 10 step 4, Decision 5 (only the lock holder rotates `dobby.out`): the rotation can run before the lock is held.**
- Step 4 reads "It sets `deps.Locked` to register the dobby ... and then ... `dobbyengine.RotatePastBound(geom.OutPath)` and ... `reopenStdio`".
- This sits in `runDobbyWith`'s sequence before step 5's `Run`. So a process that loses the lock, such as a second spawn, also rotates.
- That is the two-renames race Decision 5 rules out.
- The card does not say the rotation runs inside `Locked`, after `Run` holds the lock. The guard "only the lock holder rotates" is not placed where the lock is held.

**MEDIUM — Card 9, Decision 6 (heartbeat on the first tick ten minutes after the last): the heartbeat sits in `drivePasses`, which some ticks never reach.**
- `daemon.tick` skips `drivePasses` while an anchor move stands (`internal/dobbyengine/loop.go`, the `if moving == ""` block).
- It also leaves `drivePasses` early when `Deps.Runs` fails ("list the runs").
- So a live dobby writes no heartbeat during a standing move or a failing-runs episode, and reads as dead.
- Decision 6 names the first tick past the interval, not the first tick that drives passes.

**LOW — Card 8, Decision 1 (worktree-gone exit at the top of the loop): the binary check runs before the gone test.**
- In `daemon.loop`, `d.deps.CheckBinary()` runs after the tick and before the next iteration's top, where `worktreeGone` is tested.
- If a beam's worktree is removed during a tick while the binary has changed, the dobby re-execs first.
- The new image's `placeDobby` then fails at `lymcwd.ResolveWorktree`, which has no location, so it leaves only its envelope.
- `exitWorktreeGone` never runs, so the kept `State.Tidy` agent is never stopped and the prime's `dobby.log` gets no exit line.
- The guarded site, the stop of the tidy in flight, recurs unguarded on the re-exec path. Testing `worktreeGone` before `CheckBinary` would close it.

**LOW — Card 10, `TestDobby_ReexecsOntoAReplacedBinary`: the one-minute bound depends on the file watch opening.**
- The re-exec needs two consecutive `CheckBinary` sightings, which means two ticks after the replacement.
- The card wakes the loop through watch events on the prime's runs directory.
- Where the watch does not open (WSL, a junction; `internal/dobbyengine/doc.go`, loop section), ticks come at the probe interval, 30 s, at the earliest.
- The idle backoff doubles toward a minute, so two sightings can take longer than the test's bound.
- The test's success path is not reachable within its bound there.

Further tool calls:
- `grep` of `internal/state/state.go` for `ReadJSON` and `MkdirAll`: my context had the decision's claim that every state access creates the directory, but not the code, and the `ReadJSONNoMkdir` alternative.
