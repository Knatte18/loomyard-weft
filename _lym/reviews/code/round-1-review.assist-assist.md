---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/fabricengine/bolt.go:301
    summary: Pause admission remains inside shared lock lifetimes in board pull and synchronous pair push.
    fix: Move admission into Bolt.Pull and Bolt.PullThenCommitWritten before acquiring the board write lock, retaining it until that lock is released. Also admit PushPairAnchored before acquiring its shared teardown lock and retain admission through that release. Keep nested push admissions safe, and add public-surface regression cases that probe these locks while the pause holds the call.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/tenderengine/report.go:32
    summary: The production session adapter omits missing-session pairs from skipped when no running shuttle records exist.
    fix: In tendercli.shuttlePair.Sessions, check reed session existence independently of whether ReadSessionStates returns running records. Preserve ErrNoSession wrapping and propagate other errors. Extend pause/status coverage through the real adapter with no reed session and an absent or finished-only shuttle record set, asserting the skipped slug.
    origin: lens:code-affected-set
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/fabricengine/pushdeferral_integration_test.go:160
    summary: The lock regression never runs the competing pair's batten driver required by the acceptance criterion.
    fix: Extend the existing pause-lock regression through the batten driver path. Keep the first call paused while another pair's driver reaches and acquires the same contested push lock, assert its progress, then lift the pause and assert the first call completes. Use controlled synchronization rather than production sleeps.
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: scope
    location: internal/tenderengine/resume.go:479
    summary: Revive gains missing-session recovery beyond the requested pause and status changes.
    fix: Adapt Revive to the helper signature while preserving its prior missing-session refusal and stopping before any revival sends. Keep skip-and-continue behavior in Pause and Status only; retain the underlying error information needed by Revive rather than reconstructing an error from a rendered note.
    origin: lens:code-scope
  - id: F5
    severity: MEDIUM
    class: consistency
    location: internal/fabricengine/coalesce_test.go:144
    summary: Existing pushed-side flag omission assertions were removed without the required Loosened-Check trailer.
    fix: Record a Loosened-Check trailer on a review-range commit naming the removed CodePushed and BothPushed assertions and a measured cause. Measure the gate/deferral ordering under the changed implementation and state that evidence; an explanation of intent alone does not satisfy the rubric.
    origin: lens:code-tests
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/fabricengine/pull.go:503
    summary: Independent replay, takeover and rejected-board-push admission paths lack pause-lock regression coverage.
    fix: Add rows to the existing public-surface tests for replay, takeover and rejected board push that hold the pause and probe the relevant lock before admission. Add a case where the pause starts after acquisition and assert the operation finishes and releases its locks without waiting at an inner network gate. Cover the board-pull gap with F1's regression rather than duplicating it.
    origin: lens:code-tests
---

### [BLOCKING] Admit before the outer shared locks

**Location:** internal/fabricengine/bolt.go:301; internal/fabricengine/pendingflags.go:152

**Issue:** The new admission protects the push lock but starts after other shared locks are held.
`Bolt.PullThenCommitWritten` acquires the board write lock at line 215 and calls `pullFirst` at line 223.
`Bolt.Pull` follows the same order at lines 273–278.
A covering pause therefore keeps the board write lock held throughout its wait.
An uncovered board sync takes its absorbing push lock, then waits in `boardengine.commitDirty` for that write lock.
That transfers the pause's blockage to another pusher.

`PushPairAnchored` likewise takes its shared teardown lock at `pushanchored.go:173`.
It retains that lock when `coalesceFlagged` reaches the new wait at line 152.
The pair's exclusive teardown cannot proceed during the pause.
The landing summary acknowledges this choice, but the Sketch supplies no exception to its shared-lock requirement.
These are one ordering defect: admission must precede the outermost affected shared lock.

### [MEDIUM] Missing sessions can disappear from the report

**Location:** internal/tenderengine/report.go:32

**Issue:** The new interface contract requires a missing reed session to return `ErrNoSession`.
The engine's skip branches depend on that sentinel.
However, `shuttlePair.Sessions` returns early at `tendercli/deps.go:406` when there are no running records.
It never reaches `Live`, which probes reed.
`ReadSessionStates` returns no readings for an absent root or records whose outcomes are no longer running.
Such a pair appears in neither `sessions` nor `skipped` in pause/status output.
The added fake-pair tests always inject the sentinel and cannot expose this adapter gap.

### [MEDIUM] The competing driver is absent from the lock test

**Location:** internal/fabricengine/pushdeferral_integration_test.go:160

**Issue:** The Sketch requires another pair's batten run to take the lock while the first call waits.
The sleep callback instead invokes `pushLockFree`, which tries and immediately releases the lock directly.
This proves that the selected lock is free at the callback, but exercises no second pair or driver path.
It cannot expose an outer shared lock that prevents the competing driver from reaching the push.

### [MEDIUM] Revive changes beyond the requested verbs

**Location:** internal/tenderengine/resume.go:479

**Issue:** Goal 2 names pause and status.
Revive now converts a missing-session refusal into a skipped note and continues to later sessions and orch revival.
Adapting the helper's return values is forced; this externally visible behavior change is not.
The record has no amendment authorizing it, and the commits carry no applicable departure trailer.

### [MEDIUM] Removed assertions lack the required explanation

**Location:** internal/fabricengine/coalesce_test.go:144

**Issue:** The old table asserted that already-pushed sides did not regain pending flags.
The diff removes its `CodePushed` and `BothPushed` cases and those negative assertions.
Moving deferral earlier explains the intended contract change.
Nevertheless, the rubric requires a `Loosened-Check:` trailer naming removed assertions and a measured cause.
No commit in the reviewed range carries one.

### [MEDIUM] Other admission paths remain untested

**Location:** internal/fabricengine/pull.go:503; internal/fabricengine/takeoverwrite.go:173; internal/fabricengine/pushclassified.go:811

**Issue:** The new fabric regression reaches `coalesceFlagged` through `CoalescePushBothAt`.
The board CLI regression reaches `coalescePush` through sync and counts waits.
Neither reaches the independent admission paths for replay, takeover or rejected board push.
Removing their new admission calls would leave these regressions unchanged.
The fabric tests also do not pin admission's lifetime when a pause begins after lock acquisition.
Underlying admission tests establish the mechanism, not these callers' acquisition and release boundaries.

## Rejected

- The tests lens's revive-coverage finding is superseded by F4, which restores revive's prior behavior.
- Its ordinary session-list error finding concerns unchanged propagation; the new branch matches only `ErrNoSession`.
- Its BLOCKING severity for the remaining lock-test gaps is reduced to MEDIUM; no cited PATTERN mandates per-site tests.
- Missing dobby implementation is not a defect: `DobbyRuns` obtains the act verdict and `planPass` rejects waits.
- Existing `TestPlanPass` pause-cover rows supply that coverage; repeating them would add no behavior.
- Ungated origin reconciliation performs local work on the inspected path, so it establishes no pause wait under its locks.
- `PushWeft` has no production caller in the tree; its legacy `PushCoalesced` path establishes no reachable gap here.

## Validation

Reviewed the committed range `72c3e192e7b9afff71558513de71d385538ff8cf..HEAD` and its commit messages.
The board entry matches the Sketch's goal, with no added amendment.
The changed test waits use injected callbacks or channel synchronization rather than production delays.

Attempted `lym gate test --wait --tier 0 ./internal/tenderengine ./internal/boardcli ./internal/dobbyengine`.
The gate could not create its holder record under the hub's read-only `_bolt/.lym/gate` directory.
No tests ran in this review session; this infrastructure refusal is not a finding about the diff.
