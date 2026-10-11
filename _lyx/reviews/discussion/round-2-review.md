---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:100
    summary: D5's freshness window compares written_at against history At, which shed stamps at whole-second precision, and leaves written_at's precision and the comparison unstated, so a sub-second written_at rejects every genuine record and accepts a stale one
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:97
    summary: D5 suppresses only the blocked notice, but its goto leaves the child paused at the target, and the InnerRun notice step sends a state-changed notice for any non-running state, so a resume delayed toward the cap wakes the orch with a paused notice naming lyx loom resume
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:67
    summary: D2's trust anchor is PR-Gate approval, but PR-Gate returns Done with no review when the parent is outside require_pr_to_base, and D1's prime check does not consult that list, so a landing into an unlisted prime branch deploys unreviewed task code
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:123
    summary: D6 returns exhausted retries as a plain push failure, which turns pushRebaseFreeLogged's warn-and-leave-unpushed handling of a remote cannot-lock-ref into a propagated error, a caller-visible change D6 does not state
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:15
    summary: Scope says the retry covers every fabric push, while D6 excludes the leased update and both remote-branch deletes
---

### [MEDIUM] D5's freshness window has no stated time precision

**Location:** `_lyx/discussion/decision-record.md:100`

**Issue:** D5 accepts a record whose write time is later than the child's last history entry before the halting attempt and no later than the halting Publish entry.
The halting entry's `At` comes from `nowRFC3339` in `internal/shedengine/run.go`, which formats `time.Now().UTC()` with `time.RFC3339`, whole seconds.
Publish writes the record inside the producer call (`recordFailure` in `internal/landingshed/verifygate.go`), milliseconds before shed stamps the entry, so the two almost always share a second.
If `written_at` carries sub-second precision, a genuine record (for example 12:00:05.7) is later than its halting entry (12:00:05) and is rejected, so the automatic retry never fires.
The same truncation reverses the lower bound: after `lyx loom resume` of an earlier Publish verify halt at the same HEAD, a second attempt that halts for another reason (a push rejection, a dirty tree) leaves the first record in place, since only a passing Publish clears it.
Its write time (12:00:05.7) is later than the first halt's entry (12:00:05), the last entry before the second attempt, so the stale record reads as fresh, and batten sends a push-rejected child back to Webster-Burler.
Publish's verify Stuck is not `BudgetExempt`, so the two Publish entries do not fold, and the decision's lower bound holds only when both times share one precision.
With whole-second `written_at`, the strict lower bound also rejects a genuine record from a verify that fails within the same second as the previous entry; that direction fails safe, to the blocked notice.

**Fix:** State in D5 that `written_at` uses the same RFC 3339 whole-second UTC form as shed's history `At`, or that the comparison truncates both to one precision, and state which bound is inclusive.
Add an acceptance criterion with a record and its halting entry in the same second, and a stale record from an earlier same-HEAD halt written in the same second as that halt's entry.

### [MEDIUM] D5's goto leaves a paused child whose notice reaches the orch

**Location:** `_lyx/discussion/decision-record.md:97`, `_lyx/discussion/decision-record.md:107`, `_lyx/discussion/decision-record.md:110`

**Issue:** The goto persists the child as paused at the target (`internal/shedengine/goto.go` writes `StatePaused`), and the resume follows only once the driver parks, retried until D5's time cap.
`noticeCondition` in `internal/battenshed/notice.go` returns `noticeStateChanged` for every non-running state, and `stateChangeReady` lets it go once a stop report dated after the episode start exists, the driver strand has ended, or `noticeReportWait` has passed.
A paused episode starts at the goto, so the blocked halt's stop report does not qualify, but a resume delayed past `noticeReportWait` sends a `paused at <target>` notice.
D5 suppresses only "the blocked notice for that halt", so the orch is woken by the halt D5 exists to keep from it, and the paused notice's way forward invites the orch to resume the child while batten is still retrying the same resume.
The cap's fallback has the same gap: past the cap after `goto-done`, the child is paused, not blocked, so "sends the blocked notice with the last refusal" names a state the child is no longer in.
Separately, the InnerRun wait returns on every child state change, so the blocked→paused change ends the Call and the paused arm, documented as never resuming, must resume on `goto-done` on every entry and not only after a restart as line 110 says.

**Fix:** State in D5 that the notice suppression covers the child's paused state at the target while the marker reads `goto-done`, and that past the cap the notice batten sends names the child's actual state and the retry's refusal.
State that the paused arm resumes on `goto-done` on every Call, not only on restart.

### [MEDIUM] D2's PR-Gate trust anchor does not hold outside `require_pr_to_base`

**Location:** `_lyx/discussion/decision-record.md:67`, `_lyx/discussion/decision-record.md:71`

**Issue:** D2 makes PR-Gate approval of the landed tip the reason the deploy command may run task-authored code, and its Bound says the pull left the prime at a PR-Gate-approved tip.
`PRGate.Call` in `internal/landingshed/prgate.go` returns Done with no GitHub call and no review when the parent branch is not in landing config's `require_pr_to_base` (decision 1 of its table), and Publish is a no-op in the same case (`internal/landingshed/doc.go`).
D1's prime check compares the child's parent with the prime's checked-out branch and never consults that list.
A hub whose prime sits on a branch outside `require_pr_to_base`, such as a `master` trunk under the template's `["main"]`, therefore deploys unreviewed task code, which can edit `tools/deploy` and push outward, and D2's stated bound is false for it.
This hub lists `main`, so it is unaffected today; the key is repo-generic by D2's own rationale.

**Fix:** Make the Land row's deploy also require the child's parent branch to be in `require_pr_to_base`, skipping the deploy with a logged reason otherwise, and say so in D2's Bound and D1 step 3, with an acceptance criterion for a landing into an unlisted prime branch.
Alternatively, state in D2 that the trust anchor holds only for listed branches and that setting `deploy` on a hub whose prime branch is unlisted runs unreviewed code.

### [MEDIUM] D6's exhausted-retry error changes a warning into a failure for coalesced pushes

**Location:** `_lyx/discussion/decision-record.md:123`

**Issue:** Today a remote "cannot lock ref" refusal holds "rejected", so `PushRebaseFree` maps it to `ErrPushRejected`, and `pushRebaseFreeLogged` in `internal/fabricengine/coalesce.go` turns that into a Warn and a nil return, leaving the commits unpushed per fabric's contract.
D6 returns the exhausted case as a plain push failure, never `ErrPushRejected`, so the same refusal now propagates as an error out of the coalesced push both-sides path, the detached push child of `fabric sync` and the verbs on it.
In Publish the change only rewords the Stuck reason, from the divergence text to `push failed: ...`, but in the coalesce path it changes the outcome.
D6's Bound says a permanent refusal "fails" after the cap without saying that it used to be a warning for these callers, or whether that change is intended.

**Fix:** State in D6 what each caller class gets when attempts run out: Publish's Stuck, and for the coalesced path either the new error (with its rationale) or a Warn that leaves commits unpushed as today.
Add the chosen behaviour to the push-retry acceptance criterion.

### [LOW] Scope overstates D6's coverage

**Location:** `_lyx/discussion/decision-record.md:15`

**Issue:** Scope says the retry covers a remote "cannot lock ref" refusal "on every fabric push", while D6 excludes `UpdateRemoteBranchLeased` and both remote-branch deletes.

**Fix:** Reword the Scope line to the plain, never-forced pushes D6 names.

## Focus verdicts

- D5 failure-record freshness: the write-time window can accept a genuine record and reject a same-HEAD stale one, but only if `written_at` and history `At` share a precision, which D5 leaves unstated (F1).
- D5 retry loop: the time cap bounds the retries, and past it the orch gets a notice; until then the orch sees nothing for a held lock or an unparked driver, and the driver-busy notice for a busy pane, which is an accepted bound and not a defect.
  The notice for the child's paused state is not suppressed (F2).
- D2 trust bound: D2 now names what the command can do (run the landed tip's code, push `prod`), but the trust anchor it names fails outside `require_pr_to_base` (F3).
- D6 push retry: D6 names the covered primitives (`Push`, `PushCoalesced`, `PushRebaseFree`, Add's pushes, the archive tag push), excludes the leased update and deletes, and orders the check before `rebaseRetryTriggers` and the `ErrPushRejected` mapping, which settles coverage and ordering; what surfaces after the cap is stated, but not its effect on `pushRebaseFreeLogged` (F4).
- D4 failure episodes: an episode is one entry of the row, ended by a halt that shed persists, and a crash before that halt continues it; this distinguishes a deliberate `lyx batten run <slug>` after a halt from a crash-restart mid-notice.
  Not a defect.
