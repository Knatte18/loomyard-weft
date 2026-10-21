---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/replay.go:424-437
    summary: "replayWeft resets the weft branch onto origin before it tags the set-aside commits, so a tag failure leaves them off the branch and untagged"
    fix: "In replayWeft, create and verify the set-aside tags (tagSetAside over outcome.SetAside) before the cherryPickOnto call, or when tagSetAside fails restore the starting tip with repo.ResetKeep(startTip) before returning the error. Check replayBolt for the same order and fix it the same way."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/pushclassified.go:233-235
    summary: "A push failure that matches no class is returned unrecorded and unnotified, and the detached child's pushSideDetached returns it as a plain error, so that path is logged only"
    fix: "In classifyFailure, record any push error from a push that was run under a catch-all class (for example transport or a new unknown class) with a way forward, so every failed push returns a *PushFailedError. Alternatively have pushSideDetached record and notify through RecordPushFailure before it returns the default-case error. Add a row to TestPushFailureClasses for it."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/stuck.go:61-75
    summary: "reconcileVerdict acts on the first non-active entry of the run-scope report with no slug filter, so a stopped or ended parent pair entry halts this run"
    fix: "Give reconcileVerdict the run's slug (or have the runReconciler closure in internal/loomcli/landingdeps.go filter report.Entries to the run's slug) and skip every entry whose Slug differs, as battencli's resumeRefusal does. Add a row where the report carries a stopped parent entry and an active own entry and the verdict proceeds."
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:code-pattern-fit
    location: PATTERN.md:11-132
    summary: Commits a25fb145 and ab620b30 rewrite 14 PATTERN entries the declared changes do not list, dropping wording such as the edit-directive orch exemption and never embedded
    fix: "Revert every PATTERN.md entry except PATTERN-origin-reconcile and PATTERN-push-failure-visible to its text on main (git show main:PATTERN.md), keeping the new pattern/PATTERN-recipe-setting-lookup.md only if the long entry text is restored to match it. Rewording an entry needs its own declared change."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/replay.go:416-424
    summary: "replayWeft adds the paths of set-aside commits to touched, so a dirty file under an ended run's directory refuses the replay over a commit that is never replayed"
    fix: "In replayWeft, append detail.Paths to touched only in the branch that appends sha to kept, after the endedRunSlug check."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/destroy.go (cherryPickOnto, the ResetKeep error)
    summary: Any ResetKeep failure in cherryPickOnto is reported as ErrReplayDirty, so a missing object surfaces as a dirty stop with no paths
    fix: "In cherryPickOnto wrap the ResetKeep error in ErrReplayDirty only when git refused over a local change; return other errors unchanged."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/pushclassified.go:133-139
    summary: "When the records retry fails for a reason that is neither transport nor a conflict, the record keeps only the original rejection, so its way forward hides the real cause"
    fix: "In pushClassified, record errors.Join(pushErr, retryErr) as the failure cause in that branch, so FailedPush.Error names the lock or fetch error."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/pull.go (Pull, settleUnmovedWeft)
    summary: A weft-side infrastructure error returns from Pull before the warp arm runs, against the doc comment's promise that the weft arm never stalls warp
    fix: "In Pull, log a warning for a weft error that is not a stop and still run pullWarpArm, joining the errors; or state the exception in Pull's doc comment."
---

## Findings

### [MEDIUM] Set-aside tags are created after the weft branch is reset
**Location:** internal/fabricengine/replay.go:424-437
**Issue:** `cherryPickOnto` resets the branch onto origin at line 425, and `tagSetAside` runs at line 435. A tag failure returns with the branch already moved and the ended run's local commits untagged, recoverable only from the reflog.

### [MEDIUM] An unclassified push failure is not recorded or notified
**Location:** internal/fabricengine/pushclassified.go:233-235
**Issue:** `classifyFailure` returns an error matching no class unchanged. In the detached child, `pushSideDetached` (internal/fabricengine/coalesce.go:63-77) returns it as a plain error, so it reaches only the child's exit and log. The declared `PATTERN-push-failure-visible` text forbids a refused push left to a log line, and the first focus question asks for no path left logging only.

### [MEDIUM] The landing rows' reconcile verdict does not filter to the run's own entry
**Location:** internal/landingshed/stuck.go:61-75
**Issue:** The doc comment says the verdict is on the run's own entry, but the loop returns for the first stopped or ended entry in the report. A run-scope report also classifies the parent pair when the parent is a pair with a batten run record, and `runReconciler` in internal/loomcli/landingdeps.go:377-392 does not filter entries.

### [MEDIUM] PATTERN entries rewritten without a declaration
**Location:** PATTERN.md:11-132
**Issue:** The declared changes list two entries. The diff also shortens 14 others and adds `pattern/PATTERN-recipe-setting-lookup.md`. Some dropped wording, such as the orch exemption in `PATTERN-edit-directive`, may not survive in the background files.

### [LOW] Set-aside commits count toward the dirtiness check
**Location:** internal/fabricengine/replay.go:416-424
**Issue:** `touched` includes the paths of commits that are set aside and never replayed.

### [LOW] Every ResetKeep failure reads as dirty
**Location:** internal/fabricengine/destroy.go (cherryPickOnto)
**Issue:** A failure unrelated to uncommitted changes is reported as a dirty stop.

### [LOW] A failed retry hides its cause in the failed-push record
**Location:** internal/fabricengine/pushclassified.go:133-139
**Issue:** The record names the original rejection and drops the retry's error, so the way forward, a reset or a merge, may not address the real cause.

### [LOW] A weft infrastructure error stalls the warp arm of Pull
**Location:** internal/fabricengine/pull.go (Pull, settleUnmovedWeft)
**Issue:** Errors from the hub location, the ended-by-origin read, the weft lock or the replay default return before the warp arm runs.

## Rejected

- Tests lens, `PATTERN-push-failure-visible` has no scan test: `TestPushFailureClasses` pins each push path's behaviour, and `(test)` marks a rule any test enforces; the one gap is F2.
- Tests lens, no assertion that local-mode `RemovePair` writes no board entry: not verified, the lens did not open `teardown_test.go`'s new rows.
- Tests lens, no test for a nil `Landing.Reconcile`: not verified, struct-literal tests built without a reconcile cover it implicitly.
- Tests lens, no named test for cleanup's failed-push printing and its spawn-not-landed change: not verified, the lens did not open the integration file's rows.
- Pattern-fit lens, `os.Remove` of the failed-push record outside `destroy.go`: the chokepoint covers destructive primitives on worktrees, branches and refs, and the record is a machine-local file.
- Affected-set lens: no findings.
- Test economy: no duplicate test was named against a covering existing test.
- Reflog helpers: `judgeBranch` and `holdBackRewound` both call `trackingReflog` and `classifyRewrite`, so card 6's helpers are reused and not duplicated.
