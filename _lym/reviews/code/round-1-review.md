---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/pauserecord/hold.go:387
    summary: One admission lets every network call of the process pass the pause gate, including calls unrelated to the lock holder
    fix: In internal/pauserecord/hold.go, either scope the admission to the lock holder (Network takes an admission handle returned by Admit, and HoldRemoteRefs passes it only to the git call it runs under the lock) or state in the doc comments of Admit and Network, and in the PATTERN-remote-refs-lock text, that the pass-through is process-wide for the length of the hold, including a GitHub call or a fetch of another repository on another goroutine.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/pauserecord/hold.go:124
    summary: The in-flight marker now spans a whole remote-refs lock hold, but the NetOp comment and the pauserecord doc still describe one operation in flight
    fix: In internal/pauserecord/hold.go change the NetOp comment, and in internal/pauserecord/doc.go the line beside the NetDir description, to say that an operation named remote-refs marks a lock hold, which lasts from before the lock is taken until its release and covers the git calls and retries made under it.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/battencli/cleanup.go:292
    summary: The new error return for a failed records-HEAD read in removeRunRecords has no test row
    fix: In internal/battencli/cleanup_test.go give cleanupStubs a committedErr field that its Committed stub returns, and add a row or assertion in TestRemoveRunRecords_CommitsTheRemovalOnlyWhileTheRecordsHeadTracksTheRun that removeRunRecords returns a non-nil error, makes no commit and leaves the scratch file in place when the read fails.
---

### [LOW] Admission is process-wide, not scoped to the lock holder

**Location:** internal/pauserecord/hold.go:387
**Issue:** `Network` returns at once while `admissions` is above zero, whichever goroutine and whichever call asks.
While one goroutine holds the remote-refs lock, a GitHub call from `githubclient/transport.go:47` or a fetch in another repository passes a pause that covers the process.
The window is one git call for `withRemoteRefs`, and for `judgeAgainstOrigin` it spans the `ls-remote` retries and their sleeps.
`Admit`'s doc says "every network call of the process", so the behavior is intended, but it widens a pause's reach beyond the lock holder the Goal's cause concerns.
It corrupts nothing and the window is bounded.

### [LOW] In-flight marker semantics changed without the docs saying so

**Location:** internal/pauserecord/hold.go:124
**Issue:** Before the diff, a `NetOp` marker lasted one network call.
`Admit` now writes a `remote-refs` marker that lasts from before the lock is taken, including the wait for another holder, until the release.
The readers at `orchengine/pause.go:201` and `tendercli/deps.go:124` match on neither the op name nor the count, so nothing breaks, but the `NetOp` comment and `pauserecord/doc.go` still say a marker is one operation in flight.

### [LOW] Failed records-HEAD read in removeRunRecords is untested

**Location:** internal/battencli/cleanup.go:292
**Issue:** The diff adds a path that returns a plain wrapped error when `deps.Committed` fails, after the run directory is already removed and before the scratch is emptied.
`cleanupSlug` skips the slug on it and the cancel reports it with a re-run way forward.
The stub's `Committed` in `cleanup_test.go:55-60` never errors, so no row would fail if the path were broken.

## Rejected

- Scope of the Goal's first idea (a cancel never waits on a step held by its own pause): not a defect.
  The Seen cycle is a step holding the remote-refs lock while it waits at the gate, and the cancel's origin reconcile waiting on that lock.
  `HoldRemoteRefs` now passes the gate before the lock, so a paused step holds no lock, and the gitrepo test pins that the lock is free while the gate waits.
  The other locks the cancel takes before it ends the step were read: the board write lock covers only a local commit, and the parent lock is already exempt through the gate's slug exemption.
- Tracking judged from status and seed only, so a tracked file elsewhere in the run directory is left uncommitted: speculative.
  Status and seed are written together at run start, and the cancel's records probe and cleanup's judge already read the same two files.
- `leaveGate()` on a failed `AcquireWriteLock` untested: the failure needs an OS lock error and the lock package has no injectable seam, so a test would add machinery for a one-line branch.
- `Admit` with no gate installed untested: the increment and decrement are balanced and the doc states the behavior.
- Release order (lock release, then gate leave) not pinned by a test: both run in one goroutine and nothing observable depends on the order.
- Overlap between the gitrepo admission test and the pauserecord admission subtest: they pin different layers (gate before lock and release dropping the admission, versus the counter's pass-through), so neither duplicates the other.
- `cancel --remove` after a plain cancel not tested end to end: the unit row "already gone and untracked" fails if the guard is dropped, and `RemoveAll` makes an empty recreated directory equivalent to a missing one.
- The cancel verb's `Long` help does not mention an already-removed records directory: it says only that the cancel removes the run's records, which stays true.
