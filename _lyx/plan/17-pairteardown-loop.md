# Card 17 — pairteardown-loop

**Create:**
- `plan:internal/pairteardown#killLoop` -> `func killLoop(pidPath, lockPath, jobName string) (bool, error)`
- `plan:internal/pairteardown#awaitRunLockFree` -> `func awaitRunLockFree(ctx context.Context, lockPath string, attempts int, sleep func(context.Context, time.Duration) error) error`
- `plan:internal/pairteardown#runLockWait` -> `const runLockWait = 2 * time.Minute`

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
- `plan:internal/proc#KillTree`
- `plan:internal/proc#TerminateNamedJob`
- `internal/pairteardown#runLockFree`

**Intent:** D11: a live loop counts as busy, and the session-end step kills the loop and its step tree before the reed session ends, so no step outlives the teardown and none starts after it begins.
`quietState` gains `loop bool`; `probeQuiet` reports not quiet while the pair's loop lock, `shedrun.LoopLock` of the task's `self` run, is held, probed through the same non-blocking try `runLockFree` uses; `busyError` names "the loop is live" as the blocker, with the way forward of pausing the run (`lyx shed pause <run-id>`) or retrying once it stops.
`EndSession`, as its first act after the quiet wait and the refusal probe, calls the new `killLoop` seam on `Teardown`: `killLoop` reads the pid file through `shedverbs.ReadLoopPIDRecord`; on Linux it kills the loop by pid only while the loop lock is held and `proc.StartTime` of that pid still equals the recorded start, then kills the in-flight child's recorded process group through `proc.KillTree`, which reaches the child's descendants though the child already died with the loop; on Windows it terminates the loop's named job through `proc.TerminateNamedJob`, which ends the loop, the child and its descendants at once; an absent pid file is tolerated and reported as nothing killed.
It then waits through `awaitRunLockFree` for the run lock to be released, polling at the existing two-second interval for at most `runLockWait`, and fails naming the holder's pid from the pid file when the bound passes, so the reed session ends only after the in-flight step is dead and before any worktree is removed; batten's zero-wait path reaches the same step.
`Teardown` gains the `killLoop` and run-lock-wait seams, wired by `New` to the production functions and replaced by the untagged test's fakes.
`teardown_test.go`'s call-sequence table gains rows: a held loop lock reads as busy and names the loop; the kill runs before the session end and after the refusal probe; a run lock still held past the bound fails naming the holder and never ends the session; an absent pid file kills nothing and proceeds.
`teardown_integration_test.go`'s scenario gains a step where a stand-in loop holds the loop lock and writes a pid file recording a child whose tree holds a grandchild, and asserts after `EndSession` that the loop, the child and the grandchild are dead and the run lock is free.
`doc.go` and `PATTERN-pair-teardown.md` state that the session end kills a live loop and its step tree first and waits for the run lock, keeping the order session end, then worktree removal.
`test-budget.yaml` raises `internal/pairteardown` from 5 to 6.

**ImpactSummary:** The quiet probe gets stricter and the session end gains a kill and a bounded wait; `Request`, `Result` and `RemovePair` are unchanged, so `fabric remove` and batten's producer need no edit.
