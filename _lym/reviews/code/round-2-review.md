---
verdict: APPROVED
findings:
  - id: F3
    severity: LOW
    class: design
    carries: F3
    origin: assist:assist
    location: internal/fabricengine/pull_integration_test.go:377
    summary: TestPull_ReplayHeldByAPauseWithoutTheWeftLocks still passes when the Admit in lockWeftForReplay is removed, because the gated fetches before and after the replay satisfy its waits-above-zero assertion and the replay itself makes no network call.
    fix: In internal/fabricengine/pull_integration_test.go TestPull_ReplayHeldByAPauseWithoutTheWeftLocks, replace the check that waits is above zero with a check that pins the replay's own admission, keeping the lock probes and the WeftPulled assertion. Either assert the exact wait count of this deterministic fixture, which counts one wait for each gated fetch of the pull plus one for the admission in lockWeftForReplay, or record in atWait whether both weft lock files exist under fabricengine.RecordsLockDirPath(fx.weftPath) with the replay not yet run and assert that one wait happened at that point. Confirm by hand that deleting the Admit and its three leaveGate calls in lockWeftForReplay (internal/fabricengine/pull.go) fails the test, then restore the code.
prior:
  - id: F1
    status: resolved
    answer: Bolt.Pull and Bolt.PullThenCommitWritten admit before acquireWriteLock, PushPairAnchored admits before acquireTeardownShared, and TakeOverRun admits before its teardown locks; pullFirst no longer admits.
  - id: F2
    status: resolved
    answer: The summary drops the false premise, cites TestPlanPass's VerdictPauseCovers rows and the starts_test.go row APauseCoveringTheFrontTaskHoldsItAndTheOthersBehindIt, and states the incident's spawn path was not found; both tests exist and assert no pass or start for a covered run.
  - id: F3
    status: open
    answer: The Bolt pull, retryBoltPush, takeover and PushPairAnchored tests each fail when their Admit is removed or moved after the lock, since gated git calls run under those locks; the weft replay test fails only on a move, not on a removal.
  - id: F4
    status: resolved
    answer: Commit 888f0da2 carries a Loosened-Check trailer naming TestPushDeferral_PutsBackTheUnpushedFlags rows CodePushed and BothPushed with the cause and the accepted re-acquire case.
  - id: F5
    status: resolved
    answer: The pushDeferral doc and doc.go now say the deferral fires at the gate before each acquire and may put back flags of sides with nothing left to push; the Info line no longer claims the commits stay local.
  - id: F6
    status: resolved
    answer: Row AMissedAcquireLeavesNoAdmissionBehind holds the push lock from another handle and asserts a later covered Network call waits once.
  - id: F7
    status: resolved
    answer: TestRevive has a row with a no-reed-session pair skipped and a stalled pair sent its line.
  - id: F8
    status: resolved
    answer: TestPause and TestStatus each have a case where a plain Sessions error fails the call with nothing skipped.
  - id: F9
    status: resolved
    answer: internal/pauserecord/doc.go names remote-refs and push-lock as operations an Admit caller marks.
  - id: F10
    status: resolved
    answer: docs/overview.md says a pair with no reed session is reported under skipped in pause and lists the skipped part of status.
  - id: F11
    status: resolved
    answer: The Revive section of internal/tenderengine/doc.go and the revive help text in internal/tendercli/cli.go both say such a pair is reported under skipped.
  - id: F12
    status: resolved
    answer: The status help says skipped lists each pair, the prime included.
---

### [LOW] The weft replay test does not pin lockWeftForReplay's admission

**Location:** internal/fabricengine/pull_integration_test.go:377

**Issue:** `PassingPauseGateForTest` makes every gated git call wait once with the hub entry still live.
`Fabric.Pull` fetches the records side before `lockWeftForReplay` and pulls the code side after its release, and both are gated remote subcommands.
`replayWeft` (`internal/fabricengine/replay.go:448`) resolves, reads commits, tags and cherry-picks, all local, so no gated call runs under the weft locks.
With the `Admit` deleted, the fetch waits still make `*waits` positive and every lock probe still reads free, so the test passes.
Moving the `Admit` after either lock is caught, since its own wait then probes a held lock; deleting it is not.
The admission's only observable effect at this site is the `push-lock` in-flight mark while the locks are held, which no assertion reads.
Severity is lowered from MEDIUM because the lock-freedom property the Goal asks for holds at this site with or without the admission; the gap is coverage only.
The assistant reviewer reached the same conclusion independently.

## Focus answers

- **Gate before the outermost lock (bolt.go, pushanchored.go).**
  `Bolt.Pull` admits after its `SkipGit` return and before `acquireWriteLock`; `PullThenCommitWritten` admits before `acquireWriteLock`; both release by `defer` after the lock's release.
  `PushPairAnchored` admits after the `SkipGit`/`SkipPush` return and before `IsPrimeWorktree` and `acquireTeardownShared`, so the inner `coalesceFlagged` admission nests and passes at once.
  `pullFirst` carries only a doc line now.
  No path in these files waits at the gate while holding the board write lock, the board push lock or the teardown lock.
  `DetachedPush` takes the teardown lock shared before `coalesceFlagged`'s `Admit`, but its gate defers rather than waits, and the deferral exits the process, which frees the lock; it checks `GateCovered` before the lock as well.
- **A test per gate site.**
  `TestBolt_PullHeldByAPauseWithoutTheBoardWriteLock` fails on removal (pullFirst's fetch then waits under the write lock) and on a move.
  `TestBolt_PushRecordedRetryHeldByAPauseWithoutTheBoardLocks` fails on removal (`repo.Fetch` and `PushRebaseFree` wait under both locks) and on a move.
  `TestTakeOverRun`'s new row fails on removal or a move, since `land`'s fetch and push run under the teardown and push locks.
  `TestPull_ReplayHeldByAPauseWithoutTheWeftLocks` catches a move only: carried F3 above.
- **Dobby claim.**
  `Pause` records its entry before `coveredSessions` lists sessions, so the premise the round-1 summary used is gone from the rewritten bullet.
  `runfsm.Act` waits a covered run, landing past the parent merge included (`!in.PauseCovers` on that case), `planPass` gives no pass to a non-act verdict, and `starts.go` checks `PauseCovers` before starting a task.
  The cited `TestPlanPass` rows (`Verdict` + `PauseCovers` + trigger + `GivesNone`) and `APauseCoveringTheFrontTaskHoldsItAndTheOthersBehindIt` exist and pin that, which meets the Done-when test for dobby.
  The window where dobby reads the record before a pause writes its entry and spawns after remains; with the gate now before every push lock, such a driver waits holding no push lock.
  Not a defect.
- **Loosened-Check and deferral docs.**
  Commit 888f0da2's trailer names the test, both rows and the cause.
  The `pushDeferral` doc, the `coalesceFlagged` doc and doc.go state that the deferral fires at the gate before each acquire and that a later acquire may put back flags of sides with nothing left to push, which matches `coalesceFlagged` admitting on each loop pass and releasing after `runHeld`.

## Rejected

- No finding of the assistant's report was rejected; its one finding is merged into carried F3, with the severity lowered as the Issue explains.
