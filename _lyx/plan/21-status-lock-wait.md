# Card 21 — status-lock-wait

**Create:**
- `plan:internal/loomcli#parentLockWaiting` -> `func parentLockWaiting(waitPath string, alive func(pid int, startTime string) bool, now func() time.Time, next func(st shedengine.Status) (string, error)) func(st shedengine.Status) (string, error)`
- `plan:internal/loomcli#TestParentLockWaiting` -> `func TestParentLockWaiting(t *testing.T)`

**Edit:**
- `internal/loomcli#reviewWaitingFor`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/parentlock#ReadWait`
- `plan:internal/parentlock#WaitRecordPath`

**Intent:**
Show a lock wait as waiting in `lyx loom status` (D12), so the orch never reads a run waiting its turn as stuck.
`parentLockWaiting` reads the wait record through `parentlock.ReadWait`.
A live record renders `<producer>: waiting for the parent lock on <branch> held by <slug> <elapsed>`, elapsed from the record's start.
An absent record, or one whose writer is dead, falls through to `next`, so a wait left by a dead step is ignored.
`reviewWaitingFor` puts it first in the chain, over `parentlock.WaitRecordPath(loomengine.LoomScratchDir(loc))` and `parentlock.ProcessAlive`.
`TestParentLockWaiting`, in `reviewstatus_test.go`, asserts the live note, the fall-through for an absent record and for a dead writer, and the read error.

**ImpactSummary:** `lyx loom status` gains one waiting note; every other note is unchanged.

**Commit:** 21: loom status: a parent lock wait reads as waiting
