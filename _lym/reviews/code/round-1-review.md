---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/destroy.go:576
    summary: the destructive removal of untracked weft files has only a happy-path test, so no test pins what it must leave alone or refuse
    fix: In internal/fabricengine/droplocal_integration_test.go add rows to TestDropLocal where an untracked ignored file, an untracked file origin's tip does not track, and an untracked file on the code branch each survive the drop. Add a gate table test in internal/fabricengine/destroy_test.go where ownedUntrackedTrackedAt refuses a target the index tracks and one the named commit does not track.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/adoptcli/deps.go:69
    summary: nothing pins that the adopt's reconcile is given PushLockWait of reconcilePushLockWait, so dropping it fails no test
    fix: Add a case in internal/adoptcli that builds the Reconcile seam of hubDepsAs over a hub whose records push lock is held briefly and asserts the reconcile waits and moves, or expose the options the seam passes through export_test.go and assert PushLockWait equals reconcilePushLockWait.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/adoptcli/doc.go:403
    summary: the Step done section still says the standing record is resumed under either flag, but a record stopped at done now restarts at reconcile
    fix: In internal/adoptcli/doc.go at the Step done section, replace the sentence "Only the local-commit flag differs, so the standing record is resumed under either." with one saying the re-run restarts at reconcile with its steps cleared, as the Resume bullet near line 172 states.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:787
    summary: the adopt entry in the overview omits the pause exemption, the push-lock wait and the done-stop restart
    fix: In docs/overview.md in the adopt entry add one sentence each saying that an adopt passes every pause, that its reconcile waits up to the push-lock bound on a records branch's push lock, and that a record stopped at done restarts at reconcile.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/originreconcilemoves_integration_test.go:533
    summary: no row holds the write lock with a nonzero PushLockWait, so a regression to waiting on any held lock passes
    fix: In TestReconcileOrigin_Moves add a row that holds a records branch's write lock with PushLockWait of 3 seconds and asserts BranchBusy with zero sleeps.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/adoptcli/worker_test.go:128
    summary: the re-judge is tested only for a stop that no longer holds; a stop that still holds on re-run and the kept paused slugs are unpinned
    fix: In TestRunWorker add a row that re-runs a held-back record whose live pair is still live and asserts the run stops again with the same held-back stop and the discard way forward, and assert in the existing re-judge row that Inputs paused slugs survive the clear.
---

### [MEDIUM] Destructive untracked-file removal is covered on the happy path only

**Location:** internal/fabricengine/destroy.go:576, internal/fabricengine/droplocal_integration_test.go:74
**Issue:** `DropLocal` now deletes files, bounded by `untrackedOnPaths`, the records-side guard and the `ownedUntrackedTrackedAt` gate. The one new test shows a tracked-at-origin file removed. No test shows the ignored, origin-untracked, code-branch and index-tracked cases left alone or refused, which are the cases that keep an operator's file safe. I ran `TestDropLocal` through the tier 1 gate and it passes.

### [MEDIUM] The adopt's push-lock bound is unpinned

**Location:** internal/adoptcli/deps.go:69
**Issue:** The engine row sets `PushLockWait` itself, and the adopt tests use fake reconcile seams, so removing `PushLockWait: reconcilePushLockWait` from the adopt's reconcile leaves every test green and brings back the first note's `main-weft (busy)` stop.

### [LOW] Step done section of the adopt doc is stale

**Location:** internal/adoptcli/doc.go:403
**Issue:** It says the standing record is resumed under either flag. A record stopped at done now restarts at reconcile with steps cleared, as the Resume bullet at doc.go:172 says, so the two passages disagree.

### [LOW] Overview adopt entry misses the three new behaviours

**Location:** docs/overview.md:787
**Issue:** The entry describes the worker's steps in detail and says nothing of the pause exemption, the push-lock wait or the restart of a done-stopped record.

### [LOW] No test holds the write lock with a nonzero wait

**Location:** internal/fabricengine/originreconcilemoves_integration_test.go:533
**Issue:** `waitRecordsLocks` must return busy at once when the write lock is held, whatever the bound. The new row holds only the push lock.

### [LOW] Re-judge of a stop that still holds is untested

**Location:** internal/adoptcli/worker_test.go:128
**Issue:** The focus question on a still-holding stop reaching the operator with the `lym adopt --discard` way forward has no test, and the summary's claim that the paused slugs are kept has no assertion. The code is correct on reading: the clear leaves `Inputs` alone and the fresh reconcile re-judges.

## Rejected

- Correctness lens finding 1, `plan.onto` as a revision for `FileAtRevision`: `DropLocal` already hands the same value to `ResetKeep`, which also requires a SHA, and `TestDropLocal` passes through the real gate.
- Correctness lens finding 2, a refusal for a file origin deleted: it needs a path tracked in local HEAD yet untracked in the index, and the failure is safe, with the tag kept and nothing wrongly removed.
- Correctness lens finding 3, glob characters in a file name given to `PathTracked`: the effect is a false refusal and is safe.
- Code-tests lens finding 2, the production `ExemptPause` wiring untested: each is a one-identifier assignment that a test with a fake cannot meaningfully pin, and the behavior it carries is covered by the pause package.
- Holistic: the decision record offers either re-judging or naming `lym adopt --discard`; the diff takes the first, which the amended entry permits. The exemption covers the adopt's own process only and leaves `PATTERN-dobby-runs` intact. No `Pattern-Departure` trailer or declared PATTERN change exists to judge. The summary's claims match the diff.
