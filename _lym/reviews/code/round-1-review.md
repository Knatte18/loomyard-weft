---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/fabricengine/pushlockholder.go:55-76
    summary: The Goal asks to clear a lock whose holder is dead, the diff clears nothing and the change description records the omission as its own design decision, which the decision record does not carry
    fix: Leave the code in place and defer this finding to the run's parent. The sound change is the parent's to make, recording the omission or refusing it with lym loom decision add, and it lies outside the fixer's write surface.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/landingdeps.go:350
    summary: The landing push retry sleeps through time.Sleep for up to 3.5 minutes, so a stop or cancel arriving during a pause is not seen until the pause ends
    fix: In pullRequestDepsGuardedBy, make the PushTask retry sleep through a cancellable wait tied to the run, such as the sleepContext already wired as Sleep in the same struct, and keep the injectable sleep parameter of pushRidingOutBusyLock so the test still passes a recorder. If the PushTask seam carries no context, leave the code and say so in the fix report.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/pushlockholder.go:64-75
    summary: A holder record that decodes with no pid or no since, such as an empty object or null, is reported as held by pid 0 for an absurd age
    fix: In describePushLockHolder, after the unmarshal, treat a record whose PID is not positive or whose Since is the zero time as unreadable and return the unreadable clause, and add one row for the empty object to TestDescribePushLockHolder.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/pushlockholder.go:72-74
    summary: The dead-pid clause asserts the current holder has not written its record yet, a cause the code cannot know, since a failed record write, a pid owned by another user and a reused pid all read the same
    fix: In describePushLockHolder, reword the dead-pid clause to say the record names pid N from the given age ago, that no running process by that pid was found, and that the record may be stale, and drop the claim about the current holder. Update the DeadHolder row's want text and the doc comment to match.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/originreconcilemoves.go:94-97
    summary: The holder record written by tryRecordsLocks on a weft and by lockWeftForReplay has no test, and the change description claims both
    fix: Add a row or assertion to an existing test of each site, or one table-driven test over both, that takes the lock, asserts the record names this process while it is held and is gone after the release, and for tryRecordsLocks asserts the Bolt branch writes none.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/pushlockholder.go:44-50
    summary: The Warn-and-continue paths of holdPushLock, a record that cannot be written or removed, are untested although its contract says the lock is still held and released
    fix: Add a row to the holder test that passes a lock directory whose record path cannot be written, then asserts the returned release still frees the lock, so a regression that skips the release on a write error fails.
---

### [MEDIUM] Clearing a dead holder's lock, asked by the Goal, is neither done nor recorded

**Location:** internal/fabricengine/pushlockholder.go:55-76
**Issue:** The Goal ends with "clear a lock whose holder is dead". The diff clears no lock.
The change description gives a reason: `fabric.push.lock` is a gofrs/flock file lock (`internal/lock/lock.go`), so the kernel frees it when its holder exits and a held lock always has a live holder.
That reasoning holds for the lock itself, and the stale holder record is overwritten by the next holder, so no correct code path is missing.
The omission still departs from an explicit ask of the amended entry, and the decision record carries no `Added after Discussion` entry for it, so it is an unrecorded parent correction.

### [LOW] The landing push retry cannot be interrupted during its pauses

**Location:** internal/loomcli/landingdeps.go:350
**Issue:** `pushRidingOutBusyLock` is called with `time.Sleep`, so each of the 30 s, 1 m and 2 m pauses runs to its end whatever happens to the run.
The same seam struct wires `Sleep: sleepContext` for its other waits.
The retry itself terminates, retries only the busy-lock error and ends with the last busy error, which still names the holder, so the run still blocks as before when the lock stays busy.
The `PushTask` seam is `func() error` and carries no context, so the fix may need one to be threaded in.

### [LOW] An empty or partial holder record is reported as a live holder

**Location:** internal/fabricengine/pushlockholder.go:64-75
**Issue:** A record `{}` or `null` decodes to pid 0 and a zero `Since`.
`proc.IsAlive(0)` signals the process group and succeeds on Linux, so the report reads `held by pid 0 for 2562047h47m16s`, a wrong verdict stated with confidence.

### [LOW] The dead-pid clause states a cause the code cannot know

**Location:** internal/fabricengine/pushlockholder.go:72-74
**Issue:** The clause says the current holder "has not written its own yet".
That is false when the current holder's record write failed, which `holdPushLock` only logs at Warn, and when `proc.IsAlive` reads a live process of another user as dead because `Signal(0)` returns EPERM.
A reused pid in a stale record reads the other way, as `held by pid N`.

### [LOW] Two of the four holder-record sites have no test

**Location:** internal/fabricengine/originreconcilemoves.go:94-97
**Issue:** `TestCoalesceFlagged_HolderRecordLivesWhileTheLockIsHeld` drives only the coalescing holder loop.
Removing `holdPushLock` from `tryRecordsLocks` or `lockWeftForReplay` (`internal/fabricengine/pull.go:500`), or adding it on the Bolt branch, fails no test, though `_lym/landing/summary.md` lists both sites as covered.

### [LOW] The record write and remove failure paths of holdPushLock are untested

**Location:** internal/fabricengine/pushlockholder.go:44-50
**Issue:** The contract is that a record that cannot be written or removed is logged and the lock is still held and released.
No test makes either call fail, so a regression that returns early or skips `held.Release()` on a write error passes.

## Rejected

- A missing test for the `PushTask` closure wiring in `pullRequestDepsGuardedBy` (lens code-tests, finding 3): the closure only delegates to `pushRidingOutBusyLock`, whose five rows pin the behavior, and a test of the wiring would need a real push lock and pair through `PushPairAnchored`, a cost the public surface's tests already bear.
- The package-level `landingPushRetryPauses` slice against `PATTERN-test-speed`'s "time is injectable": the slice is passed to `pushRidingOutBusyLock` as a parameter and the test injects its own pauses and sleep, so no test waits out a production duration or mutates the variable.
- `PATTERN-tolerant-records` for the holder record written through `fsx.AtomicWriteBytes` rather than `internal/state`: each holder replaces the record whole and never reads and rewrites it, the decode refuses no unknown key (the `unknown` key in the DeadHolder row pins it), and `pendingflags.go` writes its JSON flags the same way.
- The `os.Remove` in `pushlockholder.go` against `PATTERN-fabric-git`'s destruction chokepoint: the destructive guard's allowlist is the sanctioned route, `pendingflags.go` and `index.go` are admitted the same way, and the new entry covers a file whose only removal is the holder record the same function wrote.
- A missing holder record on the Bolt `board.push.lock`: the Goal and the busy error concern `fabric.push.lock`, and the Bolt lock never returns `ErrPushLockBusy`.
- The negative age of a record with a `Since` in the future after a clock step: cosmetic and bounded to the busy report.
