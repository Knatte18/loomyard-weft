---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 (marker bullet and time-cap bullet)
    summary: D5's retry marker has no state for a retry that ran out its time cap, and the cap's start is unstated, so a later halted-arm entry restarts a capped retry and the unfinished-retry suppression outlives the cap's notice
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Scope (In bullet 4, Out bullet 4) vs D6
    summary: Scope names only the cannot-lock-ref refusal and says no other halt class is automated, while D6 also widens the bare (failed) retry from Add's pushes to every plain push primitive, Publish's included
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D2 (Bound) and D1 step 3
    summary: D2's bound claims the deploy runs only on a PR-Gate-reviewed tip, but the mechanism is a Land-time read of require_pr_to_base, which a list edit between PR-Gate and Land can make disagree with what PR-Gate did
---

### [MEDIUM] A capped D5 retry has no recorded end

**Location:** `_lyx/discussion/decision-record.md`, D5, the time-cap bullet and the marker bullet.

**Issue:** The marker holds `started`, `goto-done` and `resumed`, is reset only when batten seeds a fresh status, and "every entry of the halted arm ... finishes a retry it finds unfinished".
Past the time cap batten "stops retrying and sends the notice for the child's actual state", but nothing records that the cap was reached, and D5 does not say when the cap starts: at the `started` write (persisted) or at this process's first refusal (reset by a restart).
A new InnerRun Call follows every child state change (`checkChild` in `internal/battenshed/innerrun.go` ends the wait on any change of state), so a capped retry is re-entered as soon as the child moves:

- The cap passes with the goto refused, the marker at `started`, and the orch gets the blocked notice.
  The orch frees the driver and runs `lyx loom resume`; Publish's verify fails again with a fresh record.
  The halted arm finds `started` with the child blocked at Publish and repeats the goto: a second automatic retry, against D5's "once per batten run" bound, and the "after `resumed`" clause that would send the blocked notice never applies.
- The same entry also falls under "sends no notice for the halt ... while the retry is unfinished", so the second failure's notice is suppressed until a second cap expires.
- `goto-done` with the child blocked at Publish, and `started` with the child paused, are reachable after an orch action past the cap, and D5 names no behaviour for either.

The acceptance criterion on the cap pins only the notice sent at the cap, not what a later entry does.

**Fix:** Give the marker a terminal state for a capped retry (say `capped`), written when the cap's notice is queued, that counts as the one retry spent: a later halt at Publish after `capped` gets the blocked notice saying the retry already ran, exactly as after `resumed`.
State that the cap runs from the `started` write, so a restart does not reset it.
Scope "unfinished" in the suppression rule to `started` and `goto-done` only.
Add a criterion: after the cap's notice and an orch resume, a second Publish verify failure gets the blocked notice and no goto.

### [LOW] Scope understates D6's widening of the "(failed)" retry

**Location:** `_lyx/discussion/decision-record.md`, Scope In bullet 4 and Out bullet 4, against D6.

**Issue:** Scope In says fabric's retry "covers a remote 'cannot lock ref' refusal on the plain, never-forced pushes", and Scope Out says "only the two the board entry names (Publish verify failure, 'cannot lock ref') are automated".
D6 also moves the bare "(failed)" retry, today only around Add's two branch pushes (`internal/fabricengine/pushretry.go`), onto `Push`, `PushCoalesced`, `PushRebaseFree` and the archive tag push.
A bare "(failed)" refusal on Publish's push today matches `rebaseRetryTriggers` ("rejected") and halts Publish as divergence, so D6 automates a Publish halt class Scope says is out.
The covered and excluded primitives themselves agree between Scope and D6: the leased update is a forced push and the deletes are not plain pushes.

**Fix:** Name the "(failed)" widening in Scope In's push bullet, and narrow Out bullet 4 to halt classes handled outside the push retry, or drop the widening from D6.

### [LOW] D2's bound names a property the Land row does not check

**Location:** `_lyx/discussion/decision-record.md`, D2's Bound and the PR-Gate sentence, D1 step 3.

**Issue:** D2 bounds the deploy to "a PR-Gate-approved landed tip", and D1 step 3 enforces it by checking the parent branch against `require_pr_to_base` when the Land row runs.
`PRGate.Call` (`internal/landingshed/prgate.go`) reads the same hub-wide list when the gate runs and returns Done with no review for a branch outside it.
The two reads are separated by Finalize and the driver's exit wait; a `require_pr_to_base` edit in between, adding the prime's branch, makes the Land row deploy a tip PR-Gate passed without review.
The pulled tip is also the remote branch's head, which can hold commits past the landed one.
The list is the orch's own hub config, so this is a trusted actor's edit, but the bound as written claims more than the mechanism gives.

**Fix:** Either state the bound as the mechanism gives it (the parent branch is in `require_pr_to_base` when Land runs, and the remote prime branch receives code only through PR-Gate-reviewed landings or the operator), or have the Land row read the child's PR-Gate outcome and deploy only when PR-Gate reviewed rather than skipped.

## Focus verdicts

- D5 freshness window: settled, no defect.
  D5 states the whole-second UTC form shed's history `at` uses, the strict lower bound and the inclusive upper bound, and both same-second cases: a genuine record in its halting entry's second is retried, and a stale same-HEAD record in its own halting entry's second is not, since that entry is at or before the current attempt's lower bound.
  The acceptance criteria pin both cases.
- D5 notices and the paused arm: D5 states the suppressed notices (the halt and the paused state at the target), keeps the driver-busy notice, sends the notice for the actual state past the cap, and resumes on every halted-arm entry, which matches the new Call each state change causes.
  What happens after the cap is F1.
- D2 and D1 for a prime branch outside `require_pr_to_base`: handled; the pull runs, the deploy is skipped with an Info log, and a criterion pins it.
  The bound's wording is F3.
- D6 caller outcomes: settled, no defect.
  Publish's Stuck becomes `push failed: ...` (the non-`ErrPushRejected` branch of `publish.go`), the `pushRebaseFreeLogged` path keeps its Warn through its own sentinel, and the criterion carries both.
- Scope against D6: the primitive sets agree; the refusal forms do not, which is F2.
