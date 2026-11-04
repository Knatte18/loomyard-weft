---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:677
    summary: "A reopen request answered while GitHub reports the recorded pull request merged keeps and resets the record, so the next landing takes the merged verdicts and the reopened work has no pull request carrying it"
    fix: "In answerReopen, treat a merged pull request like a closed one: in the switch at prreview.go:678-694 add a merged case that sets fate to say pull request #N is already merged and makes no SetDraft, Comment or Close call, and extend the condition at prreview.go:696 to also remove the record when status.State is pullrequest.StateMerged, so the next landing opens a new pull request for the new work. Add a TestPRReview_Reopen row with Status returning StateMerged that asserts only the Status 7 call, no record left and a Stuck reroute whose reason names the merged pull request."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/pawlengine/ready.go:1073
    summary: "RemoveReady and the production Unready closure (the step that keeps the dobby from landing the beam again, committing even when no record was removed) have no test, because TestReopen fakes Unready whole"
    fix: "Add a table-driven tier 0 test of RemoveReady over t.TempDir covering a record naming the slug (removed, true), an absent record (false, nothing touched) and a record naming another slug (false, file left), added to the existing ready-record test file of internal/pawlengine. Add one tier 1 case over a real pair for the Unready closure in internal/beamcli/deps.go:142 that asserts the removal of the committed ready record is committed and that a second call with no record left still succeeds."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/landingdeps.go:841
    summary: "readReopenRequest, the only code deciding that a request belongs to the pair's own run, has no test row for no file, a request naming another run, a missing seed or a matching request carrying Close"
    fix: "Add a table-driven test for readReopenRequest in internal/loomcli (tier 0 over a temp-dir pair with a seed written through shedrun, or in wiring_beam_test.go) with rows: no request file reads (zero, false, nil); a request whose RunID differs from the seed StartedAt reads as none; no seed reads as none; a matching request reads (ReopenRequest{Close: true}, true, nil) when written with Close."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/prreview_test.go:551
    summary: "The decision record requires the reopen to stop the pr-reviewer, but no TestPRReview_Reopen row makes the session live and asserts StopStrand ran, so removing the stop breaks no test"
    fix: "In TestPRReview_Reopen make the fixture's SessionLive report the reviewed record's StrandGUID live and record StopStrand calls in the sessions fake, then assert in the draft row (and the close row) that the strand was stopped exactly once. Add a row where StopStrand fails that expects an error and the record left unchanged."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/pawlengine/beamawait.go:216
    summary: "When RemoveReopenRequest fails Beam-Await only logs and goes on, so the request naming the current run stays on disk and the next PR-Review reroutes at once, spending reroute-cap slots"
    fix: "In answerReopen (beamawait.go:207), return early after a failed RemoveReopenRequest without starting the dobby or posting the notice, and make enter report the entry as not complete so the next wake retries the removal; or, at minimum, add the stuck file path to the logged warning and to the notice. Add a TestBeamAwait row for the failed removal."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/beamcli/deps.go:151
    summary: "Unready always commits the landing directory, but git refuses a commit pathspec that matches nothing, so a repeated reopen after the removal was committed can fail when the directory holds no other file"
    fix: "In the Unready closure, call fabricengine.AnchoredPathHasContent(beam, loomengine.LandingDirRel()) before CommitAnchoredPaths and skip the commit when it reports false, returning removed with a nil error. Cover it in the Unready test of F2."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/hubgeom/dobbygeom_test.go:108
    summary: "TestDobbyGeometry checks only three of the four PawlPaths files and its comment still says three, so Paths.ReopenRequest is unasserted"
    fix: "In the map at dobbygeom_test.go:111 add the entry ReopenRequest with paths.ReopenRequest, and change the comment at dobbygeom_test.go:61 from three files to the pawl run's files."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/beamwiring.go:24
    summary: "The doc comment of beamDescribeDeps still says the refusal happens when the commit subjects are read, but the seam is now Commits and reads whole commits"
    fix: "Change the sentence at beamwiring.go:24 to say an empty parentBranch is refused when the beam's commits are read."
---

### [MEDIUM] Reopen of a landing whose pull request GitHub already merged

**Location:** internal/landingshed/prreview.go:677
**Issue:** `answerReopen` handles an open pull request (draft and comment, or close) and a closed one (record removed), but a merged one falls through the switch. Without `--close` the record is kept and reset and the run reroutes to Beam-Await. The next landing finds the recorded pull request merged, takes the merged verdicts and Finalize treats the reopened work as already landed, with no pull request carrying it. The state is reachable when the pull request is merged on GitHub inside the poll window while a request is read, or on a resume after a crash.

### [MEDIUM] RemoveReady and Unready are untested

**Location:** internal/pawlengine/ready.go:1073
**Issue:** The decision record's step 3, removing the ready flag so the dobby cannot land the beam again, is carried by `RemoveReady` and the production `Unready` closure, which commits even when no record was removed. `TestReopen` replaces `Unready` with a fake, and no test exercises `RemoveReady`'s slug check or the commit.

### [MEDIUM] readReopenRequest is untested

**Location:** internal/loomcli/landingdeps.go:841
**Issue:** `readReopenRequest` alone decides whether a request names the pair's own run. A wrong comparison makes PR-Review answer another run's request or ignore its own, and no test row covers it.

### [MEDIUM] Stopping the pr-reviewer is unpinned

**Location:** internal/landingshed/prreview_test.go:551
**Issue:** The decision record asks the reopen to stop the pr-reviewer. `answerReopen` calls `stopLiveSession`, but `TestPRReview_Reopen` asserts only GitHub calls and the record, so deleting the stop leaves every row green.

### [LOW] A failed reopen request removal leaves the request standing

**Location:** internal/pawlengine/beamawait.go:216
**Issue:** `answerReopen` logs a failed `RemoveReopenRequest` and continues. The request, which names the current run, stays, and the next PR-Review reads it as a fresh request and reroutes again at once.

### [LOW] Unready can fail on a repeated reopen

**Location:** internal/beamcli/deps.go:151
**Issue:** `CommitAnchoredPaths` is handed a directory that may match nothing once the removal is committed and no other file stands there, and git refuses a pathspec that matches nothing. `AnchoredPathHasContent` exists for this guard. The case is narrow, since a landing's description normally keeps the directory populated.

### [LOW] PawlPaths test omits the reopen request path

**Location:** internal/hubgeom/dobbygeom_test.go:108
**Issue:** The diff added `ReopenRequest` to `PawlPaths` and updated its doc to name four files, but `TestDobbyGeometry` still checks three and says so.

### [LOW] Stale comment on beamDescribeDeps

**Location:** internal/loomcli/beamwiring.go:24
**Issue:** The comment names commit subjects, which the `Commits` seam no longer reads.

## Rejected

- Reroute cap allows three reopens per PR-Review Done (lens:code-correctness F4): the summary declares it under Known limits, `lym loom resume` grants three more, and the decision record sets no bound on reopens, so no change to the diff is owed.
- `internal/loomrecipe/doc.go:16` says PR-Review's Stuck is always a Halt (lens:code-affected-set F2): the line describes the tapestry recipe, whose PR-Review has no reroute edge, so it is still true; the pawl paragraph below it was updated.
- Reopen answered on entry before the pause hold (handler): the entry path already makes GitHub reads and writes before its hold, and the doc states the order, so nothing contradicts a rule.
