---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/fabricengine/handover.go:pushOwedCodeTip
    summary: "an owed code tip is not pushed when origin has no code branch, though D2 and OriginCodeTip read an absent branch as the tip missing"
    fix: "In pushOwedCodeTip, when remoteHeadTip returns an empty originTip, push the tip to create the branch instead of returning false, nil. Use a lease that expects the branch absent, then run the same re-read of the records branch and the give-up reset (delete the branch it created) as the lagging-origin path. Add a TestHandOverTips row for an origin with no code branch."
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/beamcli/deps.go:315
    summary: "PushBeam calls PushPairAnchored with no Here and no RunSlug, so a beam pair's rejected records push replays instead of reading a takeover"
    fix: "In hubLandDeps PushBeam, pass PairPush with LockWait fabricengine.StatusPushLockWait, RunSlug slug and Here from pairteardown.ThisMachine(), or push through pairteardown.PushRun. Add a row to the takeover-read test or a beamcli test showing a rejected beam push over another machine's takeover entry hands over."
    origin: lens:code-affected-set
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize.go:landUnderLock
    summary: "RecordLanding stores the local merge commit before the parent push, and archiveHeldParentMerge later resets the parent branch past it, so the recorded landing commit is no longer on the parent branch"
    fix: "Record the landing after the parent push has landed or after archiveHeldParentMerge has reset the branch, taking the parent pair's head at that point, or call RecordLanding again from pushParent after a successful archive. Add a TestFinalize row where the parent push is rejected over origin holding the work and assert the recorded commit is the post-reset head."
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/landingshed/finalize_test.go:567
    summary: "TestFinalize_RecordsTheLandingCommitOnTheFirstTry is a redundancy candidate that pins the landed-size range and first-try order the covering tests do not assert"
    fix: "Add the comment line //testtiming:keep pins the landed-size range recorded from the pre-merge parent head and the record-before-push order on the first-try merge, directly above func TestFinalize_RecordsTheLandingCommitOnTheFirstTry."
    origin: handler
  - id: F5
    severity: LOW
    class: design
    location: internal/battencli/commitstatus_test.go:106
    summary: "TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed is a redundancy candidate that pins the state-to-status mapping its covering test never asserts"
    fix: "Add the comment line //testtiming:keep pins the board status text each producer and state maps to, and that Run-Shed and later rows write none, directly above func TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed."
    origin: handler
---

### [MEDIUM] An owed code tip is not pushed when origin has no code branch

**Location:** internal/fabricengine/handover.go:pushOwedCodeTip
**Issue:** `OriginCodeTip` reads an absent origin code branch as the tip missing, so the run waits and then gives the tip up after the grace.
D2 has the owing machine push the tip when origin's code branch lacks it.
`pushOwedCodeTip` returns `false, nil` for an empty `originTip`, so a pair whose first code push failed after its records landed loses the tip to its archive tag.
Records go out before the code side, so this window is real.

### [MEDIUM] PushBeam leaves the takeover read off

**Location:** internal/beamcli/deps.go:315
**Issue:** D2 reads takeover "for any caller of the pair push", and the plan's `names-from-the-caller` decision leaves a zero `MachineNames` to a fixture's push alone.
`PushBeam` passes neither `Here` nor `RunSlug`, so the read is off for it.

### [MEDIUM] The recorded landing commit can be the archived merge

**Location:** internal/landingshed/finalize.go:landUnderLock
**Issue:** `recordLanding` runs right after the local merge.
When the parent push is then rejected over origin already holding the task's work, `ArchiveParentMerge` tags that merge and resets the branch to origin's tip.
The commit `RecordLanding` stored is off the branch, and nothing re-records it.
The dobby that names the landing commit reads a commit the parent branch does not hold.

### [LOW] Test economy: fold or keep TestFinalize_RecordsTheLandingCommitOnTheFirstTry

**Location:** internal/landingshed/finalize_test.go:567
**Issue:** The redundancy report lists it as contained, covered by `TestFinalize_MergeInRequiredRetry`, `TestFinalize_PullRequest` and `TestFinalize_RecordLandingFailureKeepsDone`.
Those tests do not assert the landed-size range `parent0..landed1`, which this test does, so it is accepted and marked.

### [LOW] Test economy: keep TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed

**Location:** internal/battencli/commitstatus_test.go:106
**Issue:** The report lists it as contained, covered by `TestNewCommitStatusSeam_BoardWriteFailureDoesNotStopTheCommit`.
That test asserts commit and push counts only, and never the status text each state maps to or which rows write none, which this test pins.

## Rejected

- code-correctness C1 (board `done` failure returned unmarked): a producer error with no mark still reaches `shedengine`'s `s.Transient` classifier (`internal/shedengine/run.go:419-425`), so an unreachable origin is classified there; no evidence the failure is misread as a hard stop.
- code-correctness C4 (hand-over mark written before local removal, a failed removal is not retried): D2 prescribes writing the mark before the local removal, and leftover pairs are a later part's teardown.
- code-correctness C5 (takeover entry with an empty machine): adopt writes the machine name, and `PATTERN-tolerant-records` covers unknown keys, not a missing required one; no path produces the entry.
- code-correctness C6 (a deferred push for uncommitted tracked changes retries without limit): D1 holds the run at its transition while a push is owed, and the deferred record's error names the cause.
- code-tests F1 (admission has no test): `internal/statuscommit/statuscommit_test.go:86-87, 308-309, 359-360` assert admissions taken, still held, held at each push, and none while mid-merge.
- code-tests F2 (cancel output naming the re-run): `internal/battencli/cancel_test.go:953` and `:1008` assert `run "lym batten cancel some-slug" once origin answers`.
- code-tests F3 (fixer attempt counts push): the fork read no code for how the counts reach origin; the diff adds no separate commit-and-spawn for them, and a transition's push carries them, so no uncovered behavior is shown.
- code-tests F4 (display-only board write stays detached): an absence of a call is no behavior the diff adds.
- code-tests F5 (fabriccli verb passes `here`): wiring of an argument whose takeover behavior `TestRejectedRecordsPush_ReadsTheTakeover` pins at the public surface.
- code-tests F6 (beamcli `LockWait` field): the change is a signature adaptation with no behavior change; the beam takeover gap is F2.
