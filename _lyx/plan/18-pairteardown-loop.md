# Card 18 — pairteardown-loop

**Create:**
- `plan:internal/pairteardown#killLoop` -> `func killLoop(pidPath, lockPath, jobName string) (bool, error)`
- `plan:internal/pairteardown#awaitRunLockFree` -> `func awaitRunLockFree(ctx context.Context, lockPath string, attempts int, sleep func(context.Context, time.Duration) error) error`
- `plan:internal/pairteardown#runLockWait` -> `const runLockWait = 2 * time.Minute`
- `plan:internal/pairteardown#seizeLoopLock` -> `func seizeLoopLock(ctx context.Context, kill func() (bool, error), lockPath string, attempts int, sleep func(context.Context, time.Duration) error) (func() error, error)`
- `plan:internal/shedverbs#RunningStepPID` -> `func RunningStepPID(stepsDir string) (int, bool)`

**Edit:**
- `internal/pairteardown#Teardown`
- `internal/pairteardown#quietState`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.EndSession`
- `internal/pairteardown#Teardown.probeQuiet`
- `internal/pairteardown#Teardown.busyError`
- `internal/pairteardown/doc.go`
- `internal/pairteardown/teardown_test.go`
- `internal/pairteardown/teardown_integration_test.go`
- `pattern/PATTERN-pair-teardown.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedrun#LoopLock`
- `plan:internal/shedrun#LoopPIDFile`
- `plan:internal/shedrun#LoopJobName`
- `plan:internal/shedverbs#ReadLoopPIDRecord`
- `plan:internal/shedverbs#WriteLoopPIDRecord`
- `plan:internal/shedverbs#TeardownLoopID`
- `plan:internal/proc#KillTree`
- `plan:internal/proc#TerminateNamedJob`
- `internal/pairteardown#runLockFree`
- `internal/lock#TryAcquireWriteLock`
- `internal/shedrun#StepsDir`

**Intent:** D11: a live loop counts as busy, and the session-end step kills the loop and its step tree before the reed session ends, so no step outlives the teardown and none starts after it begins.
`quietState` gains `loop bool`; `probeQuiet` reports not quiet while the pair's loop lock, `shedrun.LoopLock` of the task's `self` run, is held, probed through the same non-blocking try `runLockFree` uses, which a starting loop outwaits because it takes its lock within card 12's grace; `busyError` names "the loop is live" as the blocker, with the way forward of pausing the run (`lyx shed pause <run-id>`) or retrying once it stops.
`EndSession`, as its first act after the quiet wait and the refusal probe, calls the new `killLoop` seam on `Teardown`: `killLoop` reads the pid file through `shedverbs.ReadLoopPIDRecord`; on Linux it kills the loop by pid only while the loop lock is held and `proc.StartTime` of that pid still equals the recorded start, then kills the in-flight child's recorded process group through `proc.KillTree`, which reaches the child's descendants though the child already died with the loop; on Windows it terminates the loop's named job through `proc.TerminateNamedJob`, which ends the loop, the child and its descendants at once; an absent pid file is tolerated and reported as nothing killed, a record carrying no child kills no group, and `killLoop` never removes the pid file.
Each kill is logged through `internal/logger` with the loop's pid, the child's process group or the job name, and whether anything was killed.
`killLoop` runs inside `seizeLoopLock`, which after each kill takes the loop lock through `lock.TryAcquireWriteLock` and holds it until `EndSession` has ended the reed session, releasing it then; when a loop spawned between the kill and the take holds the lock, it kills again and retries, at the existing two-second interval for at most `runLockWait`, and fails naming the loop lock when the bound passes.
Bound: while the teardown holds the loop lock, the still-live driver's `--until-stop` spawns no loop that can step: an invocation reads the held lock as a live or starting loop and waits, and a loop it spawned earlier cannot take the lock and exits within card 12's grace before any step, so no step starts once the session-end step begins.
The bound outlives the lock: once it holds the lock, `seizeLoopLock` reads the pid file and replaces it, through `shedverbs.WriteLoopPIDRecord`, with the teardown's mark, a record whose loop id is `shedverbs.TeardownLoopID` and which carries no loop process and no child, and leaves the mark in place when it releases the lock after the session end.
Card 12 treats a free lock beside the mark as a dead loop that no stop retires: every waiter, whatever id it adopted, and every later invocation spawns nothing and stops `interrupted` with cause `loop-exited`, the mark surviving each such stop and any envelope beside it, so a driver that outlives the session end can start no loop in the worktree up to the removal that deletes it.
When `EndSession` fails before the session end, the release first puts back the pid file it read, or removes the mark when there was none, so a pair whose removal failed keeps D2's dead-loop rule over a killed loop's record and can loop again once no record bars it.
It then waits through `awaitRunLockFree` for the run lock to be released, polling at the same interval for at most `runLockWait`, so the reed session ends only after the in-flight step is dead and before any worktree is removed; batten's zero-wait path reaches the same step.
When that bound passes it fails naming the run lock's holder as the pid of the run's newest unfinished step, which `shedverbs.RunningStepPID` reads from the in-flight step records under `shedrun.StepsDir`, a record with no envelope beside it, or as unknown when no step record is unfinished, as for a Go driver, which writes none, and, like the seize failure, ends with the way forward of retrying the removal once the holder ends; the loop's pid file never names the holder, since the loop and its child are dead by then.
`RunningStepPID` lives beside the in-flight record format it reads, reporting false for an absent or unreadable directory.
`Teardown` gains the `killLoop`, loop-lock-seize and run-lock-wait seams, wired by `New` to the production functions and replaced by the untagged test's fakes.
`teardown_test.go`'s call-sequence table gains rows: a held loop lock reads as busy and names the loop; the kill and the loop-lock seize run before the session end and after the refusal probe, and the lock is released only after the session ends, with the teardown's mark beside it, which `ReadLoopPIDRecord` reads back with `TeardownLoopID` and no child, whether or not a killed loop's record was there; a loop lock re-taken after the first kill is killed again before the seize succeeds; a run lock still held past the bound fails naming the pid of an unfinished in-flight record in the test's steps directory, or the holder as unknown with none, never ends the session and leaves the pid file as the seize found it, the killed loop's record or none; an absent pid file kills nothing and proceeds.
`teardown_integration_test.go`'s scenario gains a step where a stand-in loop holds the loop lock and writes a pid file recording a child whose tree holds a grandchild, and asserts after `EndSession` that the loop, the child and the grandchild are dead and the run lock is free.
`doc.go` and `PATTERN-pair-teardown.md` state that the session end kills a live loop and its step tree first, holds the loop lock through the session end, leaves the teardown's mark, which bars every later loop up to the removal and is withdrawn when the session end fails, and waits for the run lock, keeping the order session end, then worktree removal.
`test-budget.yaml` raises `internal/pairteardown` from 5 to 6.

**ImpactSummary:** The quiet probe gets stricter and the session end gains a kill, a loop-lock hold, a pid-file mark in the pair's own steps directory and a bounded wait; `internal/shedverbs` gains one reader no other caller uses; `Request`, `Result` and `RemovePair` are unchanged, so `fabric remove` and batten's producer need no edit.

**Verify:** GOOS=windows go vet ./internal/pairteardown
