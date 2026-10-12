# Card 21 — loop-lock-hooks

**Create:**
- `plan:internal/loomcli#runLockIdentity` -> `func runLockIdentity(loc *lyxcwd.Location, runID string) parentlock.Owner`
- `plan:internal/loomcli#battenWatcher` -> `func battenWatcher(loc *lyxcwd.Location, runID string) func() (int, bool)`
- `plan:internal/loomcli#readAgentStates` -> `func readAgentStates(loc *lyxcwd.Location, cfg shuttleengine.Config) func() ([]parentlock.AgentState, error)`
- `plan:internal/loomcli#loopLockExit` -> `func loopLockExit(locks *parentlock.Store, self func() parentlock.Owner, batten func() (int, bool)) func(stop shedverbs.LoopStop, st shedengine.Status)`
- `plan:internal/loomcli#loopIdleRelease` -> `func loopIdleRelease(locks *parentlock.Store, self func() parentlock.Owner, agents func() ([]parentlock.AgentState, error), bound time.Duration) func()`
- `internal/loomcli/looplock_test.go`

**Edit:**
- `internal/loomcli#loomCLI.specFor`
- `plan:internal/loomcli#loopLockOwner`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/parentlock#Store.List`
- `internal/parentlock#Store.Release`
- `internal/parentlock#Store.HandOff`
- `internal/parentlock#IdleWatch.Observe`
- `internal/shuttleengine#ReadAgentWaits`

**Intent:**
Fill the loop's exit and wake hooks for loom and darn runs, so the lock follows the loop that owns it (D5, D7).

`runLockIdentity` is the run part of a lock owner: the slug, the pair path and the resolved run id; `loopLockOwner` (card 11) builds on it.
`battenWatcher` reads the run's batten-watched marker as `driverWatched` does and returns the live batten's pid.

`loopLockExit` runs in the loop process at its exit.
It lists the store and acts on each record whose owner is this loop by run and by process (`self()`).
- When the status reads awaiting at PR-Gate and a live batten watches the run, it hands the record to that batten's process with `HandOff`, the new owner being the run's identity with the batten's pid and that pid's start time, so the lock is kept while the orch reviews.
- At every other stop it releases the record.
Both are compare-and-swap on the loop's own pid and start time, so a record batten or a newer loop owns is left alone.
A store error is logged and changes no stop.

`loopIdleRelease` runs on the loop's watch wakes.
It feeds `readAgentStates` into one `parentlock.IdleWatch` with the told `bound` and the loop's awake reading: a monotonic `time.Since` from the hook's creation, which does not advance during suspend on Linux.
When the rule holds, it releases every record the loop owns with one Warn each and resets the watch; the run keeps running, and the next lock-taking row reacquires and reroutes if the parent moved meanwhile.
A failed agent read counts as not waiting, so it releases nothing.
Bound: the release frees the lock only; it never stops or changes the run.

`readAgentStates` maps `shuttleengine.ReadAgentWaits` for the pair onto `parentlock.AgentState`.

`specFor`, when a location is wired, sets `spec.Loop.OnExit` and `spec.Loop.OnWake`.
The hooks use the store at `parentlock.Dir(fabricengine.BoardDir(hub))`, `self` as the run identity plus `proc.SelfRecord()` read at call time, and `landingCfg.IdleRelease()` as the bound.

`looplock_test.go` drives both hooks over a temporary store:
- the exit hook releases the loop's record at blocked, failed, paused, a stop condition, done and awaiting at Describe;
- it hands the record to a live batten at awaiting at PR-Gate, and releases it there when no batten watches;
- it leaves a record owned by another process of the run alone;
- the wake hook releases past the bound with no busy agent, and keeps the record below the bound, beside a busy agent and on a failed read.

**ImpactSummary:** loom and darn loops now release or hand off their parent lock at exit and release it on an idle bound; `loopLockOwner` shares the new run identity, and specFor's other fields are unchanged.

**Commit:** 21: loom: the loop releases or hands off its parent lock at exit and on an idle bound
