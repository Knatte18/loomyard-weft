---
verdict: APPROVED
findings:
  - id: F3
    severity: MEDIUM
    class: design
    carries: F3
    origin: lens:code-tests
    location: internal/fabricengine/pull_integration_test.go:377
    summary: The replay test still passes without the lockWeftForReplay admission because unrelated fetch waits satisfy its nonzero wait assertion.
    fix: In TestPull_ReplayHeldByAPauseWithoutTheWeftLocks, replace the nonzero aggregate wait assertion with an exact expected wait count for this deterministic fixture, including the replay admission, while retaining the lock probes and successful replay assertion. Confirm that removing the admission and matching releases from lockWeftForReplay fails the strengthened assertion, and that moving admission after either lock fails its lock probe. Restore production code after that verification; the other sites of prior F3 need no additional tests.
prior:
  - id: F1
    status: resolved
    answer: Bolt.Pull and PullThenCommitWritten now admit before the board write lock; PushPairAnchored admits before teardown locking.
  - id: F2
    status: resolved
    answer: The summary now cites the existing covered-run and covered-start tests and states that the incident spawn path was not found, following the prior finding's alternate resolution.
  - id: F3
    status: open
    answer: The Bolt, retry and takeover tests pin their admission sites, but the replay test still cannot distinguish a removed admission from waits at unrelated fetches.
  - id: F4
    status: resolved
    answer: Commit 888f0da2 carries the requested Loosened-Check trailer naming the removed rows and the accepted reacquisition behavior.
  - id: F5
    status: resolved
    answer: The deferral documentation distinguishes first acquisition from reacquisition, and the log describes restored flags.
  - id: F6
    status: resolved
    answer: AMissedAcquireLeavesNoAdmissionBehind probes a covered network call after a missed acquisition.
  - id: F7
    status: resolved
    answer: TestRevive covers a missing-session pair beside a stalled pair that is revived.
  - id: F8
    status: resolved
    answer: Pause and status now have negative cases asserting propagation of unrelated session-listing errors.
  - id: F9
    status: resolved
    answer: The pause package documentation includes push-lock among admission operation names.
  - id: F10
    status: resolved
    answer: The overview describes skipped pairs in pause and status.
  - id: F11
    status: resolved
    answer: Revive help and package documentation describe skipping pairs without reed sessions.
  - id: F12
    status: resolved
    answer: Status help explicitly includes the prime under skipped.
---

### [MEDIUM] Replay admission remains unpinned

**Location:** internal/fabricengine/pull_integration_test.go:377

**Issue:** The test accepts any positive number of pause waits.
The records fetch and pull precede replay locking, and the code fetch follows its release.
Those operations provide waits with both replay locks free even when the replay admission is absent.
The fixture's replay uses local git operations, which do not enter the network gate.
Consequently, deleting the admission and its releases preserves every assertion in this test.
Moving admission after a lock is detected; deleting it is not.

The evidence is the pull sequence in `internal/fabricengine/pull.go`,
the local replay in `internal/fabricengine/replay.go` and `internal/fabricengine/destroy.go`,
and the remote-only gate in `internal/gitexec/gitexec.go:248`.
This carries only the remaining replay coverage gap from F3.

The review found no BLOCKING defect, so the format's verdict is APPROVED with this MEDIUM finding.
The coverage conclusion comes from tracing the fixture and its callees; no mutation run was performed.
All lens reports are saved beside this report.
