# Card 2 — parentlock-wait-idle

**Create:**
- `internal/parentlock#Waiter`
- `internal/parentlock#ErrPaused`
- `internal/parentlock#Store.Acquire`
- `internal/parentlock#Wait`
- `internal/parentlock#NewWait`
- `internal/parentlock#WaitRecordPath`
- `internal/parentlock#WriteWait`
- `internal/parentlock#ReadWait`
- `internal/parentlock#RemoveWait`
- `internal/parentlock#AgentState`
- `internal/parentlock#IdleWatch`
- `internal/parentlock#IdleWatch.Observe`
- `internal/parentlock/acquire_test.go`
- `internal/parentlock/idle_test.go`

**Edit:**
- `internal/parentlock/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `cmd/lyx#unboundedWaitFiles`
- `pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `internal/parentlock#Store.TryAcquire`

**Intent:**
Add the queue wait, the wait record and the idle rule the rows, the loop and batten share (D5, D7, D12).

`Store.Acquire(ctx, branch, owner, w)` loops `TryAcquire` until granted, waiting with no time limit in the queue, polling from one second doubling to thirty while the holder is unchanged and back to one second when it changes (`PATTERN-daemon-wakeups`).
`Store.Acquire` lives in `internal/parentlock/acquire.go`, which `unboundedWaitFiles` gains, so the guard's sub-second scan covers this wait.
`PATTERN-daemon-wakeups.md`'s What counts list gains the parent lock wait in `internal/parentlock/acquire.go`, which polls from one second doubling to thirty.
`Waiter` carries three told seams, each nil-is-absent:
- `OnWait func(holder Record)` is called once when the wait starts and again whenever the holder changes, so the caller writes its wait record and its wait mark;
- `Paused func() bool` is read before every sleep, and true returns `ErrPaused` with the caller's ticket removed;
- `Sleep func(ctx context.Context, d time.Duration) error` replaces the timer wait, so a test never sleeps.
A cancelled context returns its error with the ticket removed.
A `TryAcquire` error, `ErrUnreadable` included, is returned at once with the ticket removed.
Bound: `Acquire` only waits and takes; it never frees another owner's live record.

`Wait` is `Branch`, `Holder` (the holder's slug), `PID`, `StartTime` and `Started`; `NewWait(branch, holder)` stamps the calling process's pid and start time and the current time, so a caller outside this package needs no `proc` import.
`WaitRecordPath(dir)` joins `parent-lock-wait.json` onto the told run directory.
`WriteWait` writes atomically, replacing any record a dead step left; `ReadWait` reports found false for an absent record and for one whose writer is not alive under the told `alive`, so a wait left by a dead step is ignored; `RemoveWait` treats an absent file as success.

`AgentState` is `Name`, `Busy` and `Waiting`.
`IdleWatch` carries `Bound` and the first-seen-waiting time per agent name, on the caller's awake clock.
`Observe(agents, read, awake)` returns true when no agent is busy and at least one has been waiting for at least `Bound`, measured from the `awake` reading at which `Observe` first saw it waiting.
An agent no longer waiting, or absent, drops its first-seen time; `read` false (an unreadable state) counts every agent as not waiting, so it clears every first-seen time and returns false.
Because the times are awake readings the caller supplies, time its clock did not see, such as a suspend, never counts as waiting.

`acquire_test.go` pins the wait: a second owner waits while the first holds and is granted after `Release`, with `OnWait` called naming the holder; the poll delays double to thirty seconds through the fake `Sleep`; `Paused` returning true yields `ErrPaused` and no ticket stays behind; a cancelled context returns its error; an unreadable record returns `ErrUnreadable` without waiting; a wait record written by a dead pid reads as absent and is replaced by the next `WriteWait`.
`idle_test.go` pins `Observe` as one table: waiting past the bound with nothing busy is true; below the bound, beside a busy agent, and with `read` false are false; an agent that stops waiting and waits again starts its bound afresh.

**ImpactSummary:** additions to the new package only; no other package calls it yet.

**Commit:** 2: parentlock: queue wait, wait record and idle rule
