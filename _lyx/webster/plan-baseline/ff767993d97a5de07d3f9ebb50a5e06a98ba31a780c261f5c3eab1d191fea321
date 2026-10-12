# Card 22 — status-lock-wait

**Create:**
- `plan:internal/loomcli#parentLockWaiting` -> `func parentLockWaiting(waitPath string, alive func(pid int, startTime string) bool, now func() time.Time, next func(st shedengine.Status) (string, error)) func(st shedengine.Status) (string, error)`
- `plan:internal/loomcli#TestParentLockWaiting` -> `func TestParentLockWaiting(t *testing.T)`
- `plan:internal/loomcli#TestAgentActivity_LiveLockWaitReadsAsActivity` -> `func TestAgentActivity_LiveLockWaitReadsAsActivity(t *testing.T)`

**Edit:**
- `internal/loomcli#reviewWaitingFor`
- `internal/loomcli#agentActivityFor`
- `contracts/specs/loom-status-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/parentlock#ReadWait`
- `plan:internal/parentlock#WaitRecordPath`

**Intent:**
Show a lock wait as waiting in `lyx loom status` (D12), so the orch never reads a run waiting its turn as stuck.
`parentLockWaiting` reads the wait record through `parentlock.ReadWait`.
A live record renders `<producer>: waiting for the parent lock on <branch> held by <slug> <elapsed>`, elapsed from the record's start.
An absent record, or one whose writer is dead, falls through to `next`, so a wait left by a dead step is ignored.
`reviewWaitingFor` puts it inside `gateWaiting`, ahead of the verify note, over `parentlock.WaitRecordPath(loomengine.LoomScratchDir(loc))` and `parentlock.ProcessAlive`.
So `gateWaiting` stays the outermost wrapper and a gate wait still never hides the rest, while a live lock wait wins over the verify, shuttle and parent-review notes, and its doc comment names the new order.
`TestParentLockWaiting`, in `reviewstatus_test.go`, asserts the live note, the fall-through for an absent record and for a dead writer, and the read error.

`contracts/specs/loom-status-spec.md`'s `## The waiting key` section counts five waits.
Its ordered list gains the parent lock wait second, after the gate wait and before the verify, read from the run's parent lock wait record, which a landing row writes while it waits for the parent lock.
Its render forms gain `<producer>: waiting for the parent lock on <branch> held by <slug> <elapsed>`, and it states that a lock wait record whose writer is dead is ignored.
Its gate-wait sentence is unchanged, and its read sources name the lock wait record beside the markers, the wait records and the round store.

A lock wait also counts as activity for the loop's step watchdog (D5), so a run queued at Merge-In behind a slow run is never killed as idle after `step_idle_timeout_min`.
`agentActivityFor` reads `parentlock.ReadWait(parentlock.WaitRecordPath(loomengine.LoomScratchDir(loc)), parentlock.ProcessAlive)` right after the verify marker.
A live record returns `time.Now(), true, nil`, as a running verify does; an absent record, or one whose writer is dead, falls through to the shuttle reads.
`TestAgentActivity_LiveLockWaitReadsAsActivity` asserts that a live wait record reads as activity now and a dead writer's record does not.

**ImpactSummary:** `lyx loom status` gains one waiting note, every other note is unchanged, and the step watchdog reads a live lock wait as activity.

**Commit:** 22: loom status: a parent lock wait reads as waiting
