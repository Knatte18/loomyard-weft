---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: The proposed branch "a spawn path ignores the pause" does not hold; the proposal's own other branch applied. Every batten spawn site was read, and each reads PauseCovers or the act verdict before it spawns, so none ignores the pause. The summary bullet was rewritten to drop the false premise, cite the TestPlanPass rows and the starts_test.go row, and say the incident's spawn path was not found.
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
---

## Changes

- **F1.**
  `Bolt.Pull`, `Bolt.PullThenCommitWritten` and `PushPairAnchored` now call `pauserecord.Admit(pushLockGateOp)` before their outermost lock, released by a defer.
  `pullFirst` lost its own `Admit`, since both callers admit first.
  The docs of `pullFirst`, `PushPairAnchored`, `Pull`, `PullThenCommitWritten` and the `doc.go` paragraph say the gate is passed before the outermost lock.
  The summary's design bullet that kept the teardown lock before the gate is replaced.
  The probe test is `TestBolt_PullHeldByAPauseWithoutTheBoardWriteLock`, with a row for each of the two methods.
- **F2.**
  Only `_lym/landing/summary.md` changed, and no code or test.
  A read-only search found three batten spawn sites, `startFirstPass` and `spawn` in dobby and `lym batten start`.
  Each reads `PauseCovers` or the act verdict first, and no log of the incident was found in this worktree.
- **F3.**
  `PassingPauseGateForTest` in `export_test.go` keeps a covering pause live and runs a probe at every wait, so a removed or moved `Admit` leaves a gated call waiting under the lock and fails the probe.
  New tests: `TestPull_ReplayHeldByAPauseWithoutTheWeftLocks` for `lockWeftForReplay`,
  `TestBolt_PushRecordedRetryHeldByAPauseWithoutTheBoardLocks` for `retryBoltPush`,
  and the `TestTakeOverRun` row `APauseHoldsTheTakeoverWithoutAnyTeardownOrPushLockAndTheEntryStillLands` for the takeover.
  `Bolt.Pull`'s site is covered by the F1 test.
  `TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock` now uses `PauseGateForTest`, a gate that lifts the pause at its first wait.
- **F4.**
  An empty commit carries the `Loosened-Check` trailer for rows `CodePushed` and `BothPushed`, with the re-acquire case stated as accepted.
- **F5.**
  The `pushDeferral` doc, the `doc.go` lines, the `coalescePushSides` doc and the `doc.go` line on a deferred push say the deferral fires before each acquire.
  The Info log now reads "exiting with its pending flags put back".
- **F6.**
  Row `AMissedAcquireLeavesNoAdmissionBehind` of `TestCoalescePushRequest_CoveredCallDefers` misses the lock from another handle, then asserts that a `pauserecord.Network` call under a covering entry waits once.
- **F7.**
  `TestRevive` row: a pair with `errNoReedSession` is in `Skipped` with the `noReedSessionNote` reason and the stalled pair is revived.
- **F8.**
  `TestPause` row and `TestStatus` subtest: a plain `errSessionsRead` fails the call and lists nothing under `skipped`.
- **F9.** The `pauserecord` doc names `remote-refs` or `push-lock` as operations that mark a lock hold.
- **F10.** `docs/overview.md` lists the `skipped` part of `lym tender status` and the pause skip.
- **F11.** The tenderengine Revive section and the revive help text say a pair with no reed session is reported under `skipped`.
- **F12.** The status help says `skipped` lists each pair, the prime included.
- **Self-check change.**
  `TakeOverRun` now admits before its teardown locks, and `takeoverBranch.land` lost its own `Admit`.
  `TestPushPairAnchored_HeldByAPauseWithoutTheTeardownLock` pins the `PushPairAnchored` site.
  The takeover row probes the teardown locks too.
  The fixes are in commits `f1bcc09d` (F1), `7552e79a` (F3), `9529cf7f` (F6), `888f0da2` (F4), `dca70944` (F5), `f654d5f1` (F7),
  `72540a0c` (F8), `ecf1ec45` (F9), `95bff162` (F10), `df7c3f41` (F11), `68c196a9` (F12) and `9d6fc450` (self-check).
  F6, F8 and F12 share files with F3, F7 and F11, so their commits are empty and say so.
- **Gate repair.**
  The gate flagged two comment lines of 120 characters or more, in `coalesce.go` (the `coalescePushSides` doc) and `doc.go` (the deferred-push line).
  Each is split at a sentence or clause boundary, with no change of meaning.
  The gate then flagged the `doc.go` line as a wrapped break, so it is now three one-sentence lines.
- **Verify.** `lym gate test` passes at tier 0 for the five touched packages and at tier 1 for `fabricengine`, which holds the new tests.

## Self-check

- fix-code-correctness, MEDIUM: `PushPairAnchored`'s outer `Admit` has no test. Fixed with my own change, `TestPushPairAnchored_HeldByAPauseWithoutTheTeardownLock`.
- fix-code-correctness, LOW: the takeover probe checks only push locks. Fixed together with the consistency finding below.
- fix-consistency, MEDIUM: `TakeOverRun` waits at `land`'s gate with both teardown locks held. Fixed with my own change: it admits before the teardown locks, and `land` lost its redundant `Admit`.
- fix-consistency, LOW: the `coalescePushSides` doc twin was not updated. Fixed.
- fix-consistency, LOW: the `doc.go` line on a deferred push said it leaves its commits local. Fixed.
- fix-proposal, MEDIUM: `PullThenCommitWritten`'s admission spans the write callbacks. Disputed, ground `every fix worse`.
- fix-proposal, MEDIUM: the doc and summary overstate that no lock is held at the gate. Fixed with the `TakeOverRun` change.
- fix-proposal, LOW: `PushPairAnchored`'s `Admit` has no test. Duplicate of the correctness finding, fixed.
- fix-affected-set, MEDIUM: `TakeOverRun` holds teardown locks at the gate. Duplicate, fixed.
- fix-affected-set, LOW: other teardown-lock takers are unchecked. Deferred, reason below.

## Disputed

- **Self-check finding, fix-proposal MEDIUM: `PullThenCommitWritten`'s admission spans each write callback.**
  Ground: every fix worse.
  `Admit` is process-wide, so a call a callback made would pass a pause.
  The only other scope is to admit around the pull alone, which leaves the board write lock held across the callbacks and the commits.
  The commits' git calls go through the same gate (`internal/gitexec/gitexec.go:254`), so a pause would hold them with the write lock taken, which is the defect F1 names.
  `BoltWrite` is a callback that writes board files and returns their paths (`internal/fabricengine/bolt.go:184`),
  and the lock is held across every write by contract (`internal/fabricengine/bolt.go:209`).
  The summary records the trade-off.

## Deferred

- **Self-check finding, fix-affected-set LOW: the teardown-lock takers in `ownerarchive.go`, `handover.go` and `remoteteardown.go`.**
  They are teardown and handover operations whose git calls wait at the gate while they hold the teardown lock; none is a push-lock gate site.
  Admitting them first changes how removal and hand-over behave under a pause, which this review does not ask for and the operator should decide.
  No finding of the review depends on it.

## Rulings

None reached me.
