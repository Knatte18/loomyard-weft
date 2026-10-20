---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D5 case selection, D6 first paragraph, D2 third paragraph, Scope Out reroute bullet
    summary: D5 compares the settle record at every Call before anything else and every archive keeps it, so a generation already under way is never judged; an amendment re-archives and re-seeds on each Call, and the verify retry's or reroute's round N+1 is discarded or skipped
    fix: In D5, state that the settle-record comparison runs only when the run directory holds no generation newer than the settled one; an amendment generation marked in D6's generation record, or a round report numbered past the settled round, takes today's round-by-round path with no comparison. In D2 and the Scope Out reroute bullet, name the mechanism that makes the settled generation unsettled when round N+1 starts (for example, the round producer's report for round N+1 on disk, or the admitted goto clearing the settled mark), so the Bouncer judges that round. Add acceptance criteria that an amendment's second Call judges round 1 without re-archiving, and the verify retry's round N+1 is judged.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D5 code-segment identity bullet, Open risks first bullet
    summary: The code-segment identity, the task diff against its merge base, includes the durable _lyx/reviews run records the status seam commits at every transition, the settle record itself and the carry-over in the decision record, so the identity always differs and case 1 never fires for Webster-Review or Darn-Review
    fix: In D5's code-segment bullet, define the identity as the task diff against its merge base restricted to paths outside the run's own records (the `_lyx` durable tree, or at least the review run root and the discussion directory), and state that rule in the bullet rather than in Open risks. Replace the Open risks first bullet with nothing, or with a pointer to D5, since the exclusion becomes a Decision. Add an acceptance criterion that a status commit and a carry-over commit between settle and re-entry leave the code identity unchanged.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D5 first paragraph and document-segment bullet
    summary: The settle record is written before carry-over, and carry-over rewrites decision-record.md, one of the Discussion segment's own subject files; later segments' carry-overs and lyx loom decision add rewrite it too, so a Discussion resume after a failed Commit and any later Discussion re-entry read as an amendment
    fix: In D5's document-segment bullet, exclude from the Discussion identity every block written by Go or by `lyx loom decision add` (the `<!-- lyx:carry-over … -->` blocks and the `### Added after Discussion` entries and bullets), or take the identity after carry-over and Approve; state which. Add an acceptance criterion that a Discussion-Bouncer whose CONVERGED settle with open findings failed at Commit re-settles on resume with no spawn, and that a later segment's carry-over leaves the Discussion identity unchanged.
  - id: F4
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D1, D6 second paragraph, acceptance criterion for a failed Commit followed by a writer goto
    summary: A Commit failure returns no outcome, so the history gains no Done; with D1 the writer goto no longer ends the episode, so the amendment starts with the prior generation's Stucks counted and halts budget-exhausted at its seed instead of getting its allowance
    fix: In D6, state where an amendment generation's count starts, either because the Bouncer writes a budget-exempt marker, or Shed treats the generation record's start as an episode end for the segment, so the amendment's Shed count and Bounces seam count start at 0. Add that marker to D1's list of episode enders and to the Constraints' tolerant-records line, and extend the failed-Commit-then-writer-goto acceptance criterion to assert the amendment gets its allowance.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D2 first paragraph and Bound, D5 case 1
    summary: D2 refuses a goto only for a settled record, so in the interrupted-settle state a goto onto the review or fix row is admitted; if its fixer changes nothing, case 1 re-settles approved over that round's unjudged BLOCKING review, and with an allowance of 0 a fix-row edit is approved unreviewed, with no stated bound
    fix: In D2, extend the refusal to a segment whose settle record exists but is not yet marked settled, with the same way forward (the Bouncer row, which re-settles, or the writer row), and state in D2's Bound what the interrupted state admits. Add an acceptance criterion that a goto onto the review row of an interrupted settle is refused.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D2 first paragraph, D5 settled mark and archive rule
    summary: The settled mark is carried through every archive and never cleared when an amendment starts, so D2 refuses an operator goto or a tackler or arbiter re-launch onto a failed review or fix row of an amendment generation that is not settled
    fix: In D5 or D6, state that starting an amendment generation clears the carried record's settled mark, or that D2's check reads the generation record and refuses only when the live generation is settled. Add an acceptance criterion that a goto onto a review row inside an unsettled amendment is admitted.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D6 second paragraph and Bound
    summary: The amendment's hard cap is the round the allowance reaches, which is round 1 at the default; atHardCap(1) makes a CONTINUE escalate with cause hard-cap offering only accept, so the Bound's claim that the escalation verbs work as in any generation and that a bigger amendment escalates for more rounds is false
    fix: In D6, set the amendment's hard cap one round past the allowance, clamped to the perch's hard cap, so a parent continue at the soft cap buys one judged round; or keep the cap and rewrite the Bound to say the parent can only accept or send the run to the writer row. State which, and add an acceptance criterion for the escalation an amendment CONTINUE at the allowance offers.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3 fourth paragraph and Bound
    summary: The hand-over rule sits ahead of the tackler and arbiter rules and a blocked stop is binaryEligible, so after a deploy the successor driver's first Step resumes the budget-exhausted halt and, with budget_capped ending the episode, grants a fresh budget with no parent
    fix: In D3, place the new rule ahead of the hand-over rule as well, or exclude a budget-exhausted stop from binaryEligible, and state it in D3's Bound. Add an acceptance criterion that a budget-exhausted halt with a changed on-disk build is handed back, not handed over.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3 fourth paragraph
    summary: The rule matches a stop whose reason starts with ReasonBounceBudgetExhausted, but a stop built from the status file carries the halt reason in Error and leaves Reason empty, so a rechecked budget-exhausted stop falls through to the tackler
    fix: In D3, state that the rule reads the stop's reason or, for a stop built from the status, the status error, as D4 does for pause_reason, and extend the acceptance criterion on the hand-back to cover a stop read from the status.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D2 third paragraph and Bound
    summary: The verify-retry bypass is keyed on the Publish failure record naming the row, which stands until RemovePublishFailure, so every goto onto Webster-Burler-Review while the record stands, from the orch or a tackler, re-runs a review over the settled segment; the bound limits the row, not the count or the caller
    fix: In D2's Bound, key the bypass on the driver's one verify retry (its settle record) or on an unspent retry, so a second goto onto that row is refused, and state that bound. Add an acceptance criterion that a second goto onto the failure record's row is refused.
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D5 Merge-Review bullet, D6 Bound, D7
    summary: A new parent merge-in changes the Merge-Review identity and becomes an amendment under the allowance, and with review_amend_max_bounces 0, settable through run_settings, every later merge-in is approved with no review; no Decision states this bound
    fix: Either exclude Merge-Review from D6, so a new merge-in is a new subject reviewed in full as today, or state in D6's Bound and D7 that a merge-in after the first gets only the allowance and that 0 approves it unreviewed. Add the matching acceptance criterion.
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4 first bullet and Rationale
    summary: Goto's --by is hidden and admits only shed, and loom's bare pause is refused by its BarePause hook, so a pause --by as goto does names no operator or orch, and an operator goto records no who either, missing D4's rationale that a paused run says who paused it
    fix: In D4, state the values the pause verbs' --by admits (a free caller label, or the existing label set) and what it bounds (a label only, it skips nothing), whether the goto's --by is widened to the same values, and that loom's bare pause stays refused so its pause reason comes from a --before or --after condition plus the label. Update the pause-reason acceptance criterion to match.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4 first paragraph, acceptance criterion on every path writing paused
    summary: D4 clears pause_reason on every Relaunch write that does not record paused, but a Relaunch with no target or an in-effect target writes a paused status unchanged, so read literally it leaves paused with an empty reason
    fix: In D4, state that a write that leaves the state unchanged also leaves pause_reason unchanged, and that only a write moving the run out of paused clears it.
  - id: F14
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D2 fourth paragraph
    summary: The tackler and the arbiter validate their verdict's goto with CheckGotoTarget before applying it, but D2's seam sits only on GotoRequest and RelaunchRequest, so a verdict naming a settled review row passes validation and is refused only at apply
    fix: In D2, state that the told seam also runs where the tackler and arbiter validate a verdict's goto, so an invalid verdict is corrected there, or state that the apply-time refusal and hand-back are the intended bound.
  - id: F15
    severity: LOW
    class: decision
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D8 last paragraph
    summary: D8 retires friction_timeout_min in bootstrap.go, but the key still bounds the Friction-Reflect row's reflection and lock wait in run.go, and bootstrap.go names it only in comments on the awaitRunLockHalted arm; the record leaves open whether the key, the arm or the comments go
    fix: In D8, say that the friction_timeout_min key stays for the Friction-Reflect row and that only the bootstrap's after-halt reflection allowance goes, naming whether the awaitRunLockHalted arm is removed or only its comments are rewritten.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D9, Goal, acceptance criterion on the fork shortfall
    summary: ErrClusterForksMissing also fires for a nil audit and for more forks than requested, so D9's retry on the sentinel and its criterion and title on fewer forks describe different triggers; a resumed reviewer that reaches done with a shortfall is accepted outside the attempt loop
    fix: In D9, state whether a surplus and a nil audit are retried like a shortfall, align the title, Goal and criterion with that, and state whether the resumed-reviewer path joins the retry.
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D3 second paragraph
    summary: D3 says resuming the halt grants one more budget and nothing else does, but D1 keeps a reroute into the segment and a Done by a segment row as episode enders
    fix: In D3, change and nothing else does to say no goto does.
  - id: F18
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3 first paragraph, Notes for the plan writer second bullet
    summary: episodeStuckCount does not stop at a reroute_capped entry today; that entry is budget-exempt and skipped, and only rerouteCount stops at it, so as it does for reroute_capped misstates the existing code
    fix: In D3, replace as it does for the reroute cap's reroute_capped entry with as rerouteCount does at a reroute_capped entry, and adjust the plan-writer note the same way.
  - id: F19
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md acceptance criterion after a PR-Gate reject
    summary: The criterion that loom's Plan-Review after PR-Rework runs an amendment generation has no exception for the exempt rework that D6 says PlanReviewSkippable skips entirely
    fix: In that criterion, add unless PR-Rework's exempt skip approves the generation without review.
  - id: F20
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D5 last paragraph
    summary: Case 1 re-runs carry-over too, but only Approve and Commit are required to be idempotent; a carry-over commit over an unchanged entry must also succeed and commit nothing
    fix: In D5's last paragraph, require the CarryOver seam to be idempotent as well.
  - id: F21
    severity: NIT
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D5 second paragraph, D6 last paragraph
    summary: D5 reads the settle record before anything else at Call entry, while D6 and today's Call run the skip seam first
    fix: In D5, change before anything else to after the skip seam.
---

### [BLOCKING] A generation under way is never judged

**Location:** `_lyx/discussion/decision-record.md` D5 case selection, D6, D2's verify retry, Scope Out reroute bullet.

**Issue:** D5 reads the settle record before anything else at every Call, and every archive carries it forward.
Inside an amendment the identity still differs from the record, so each Call lands in case 2 again, archives and re-seeds, and no amendment round is judged.
After the verify retry or the Webster reroute, round N+1 either changes the subject (case 2 archives it) or leaves it unchanged (case 1 re-settles Done unjudged), contradicting D2's "runs as today" and Scope Out's "judges it as any round".
Evidence: `internal/shedadapters/bouncer.go:328-450`, `internal/sheddriver/drive.go:286-288`.
Raised by every lens.

### [BLOCKING] The code identity includes the run's own committed records

**Location:** D5 code-segment bullet.

**Issue:** The review run root `_lyx/reviews` is committed at every transition (`internal/loomengine/config.go:271-276`, `internal/loomcli/wiring.go` RunRoot comment), the settle record lives in it, and carry-over commits `decision-record.md` (`internal/loomcli/wiring.go:395-400`).
The diff against the merge base therefore changes between every settle and re-entry, and the "unchanged subject returns Done" criterion cannot hold.
The exclusion requirement also sits in Open risks instead of the Decision.

### [BLOCKING] Carry-over changes the Discussion subject

**Location:** D5 first paragraph and document-segment bullet.

**Issue:** The settle record is written before carry-over, and the Discussion segment's carry-over then rewrites `decision-record.md`, one of its `artifact_paths` (`contracts/recipes/loom-recipe.yaml:131-141`, `internal/shedadapters/bouncer.go` `settle`).
Later segments' carry-overs and `lyx loom decision add` rewrite it too, so a resume after a failed Commit and every later re-entry read as an amendment.

### [BLOCKING] An amendment after an interrupted settle inherits the spent count

**Location:** D1, D6, acceptance criterion on a failed Commit followed by a writer goto.

**Issue:** A seam failure returns no outcome, so `appendHistory` records nothing (`internal/shedengine/run.go:274-286`), and no Done ends the episode.
With D1, the goto to Plan-Write ends nothing either, and Plan-Write is outside the segment, so `episodeStuckCount` still counts the previous generation's Stucks and the amendment's seed Stuck hits the budget block.
This re-creates the board's `ubuntu-2404-support` case.

### [MEDIUM] The interrupted-settle state admits gotos with no bound

**Location:** D2, D5 case 1.

**Issue:** D2 refuses only a settled record.
A goto onto the review row of an interrupted settle runs a round, and if the fixer disputes every finding, case 1 re-settles over an unjudged BLOCKING review, bypassing `judgmentRefusal`; with allowance 0, a fix-row edit is approved unreviewed.

### [MEDIUM] The settled mark outlives the settled generation

**Location:** D2, D5.

**Issue:** The settled mark is carried through every archive and nothing clears it, so D2 refuses gotos and re-launches inside an unsettled amendment (`internal/sheddriver/tacklerconsume.go:56`, `internal/loomcli/arbiterconsume.go:62`).

### [MEDIUM] The amendment's hard cap leaves the parent only accept

**Location:** D6.

**Issue:** At the default allowance the hard cap is round 1, and `atHardCap` (`internal/shedadapters/bouncer.go:876-880`) makes a CONTINUE escalate with cause hard-cap offering accept alone, so the Bound's "escalation verbs work as in any generation" is false.

### [MEDIUM] The hand-over resumes a budget-exhausted halt

**Location:** D3.

**Issue:** A blocked stop is `binaryEligible`, and the hand-over rule precedes the tackler and arbiter rules (`internal/sheddriver/driverules.go:168-217`).
The successor driver's first act is a Step (`internal/sheddriver/drive.go:178-190`), which resumes the halt, and `budget_capped` then grants a fresh budget without the parent, breaking D3's Bound.

### [MEDIUM] The driver rule misses a stop read from the status

**Location:** D3.

**Issue:** `stopFromStatus` fills `Error`, not `Reason` (`internal/sheddriver/driverules.go:98-109`), so a rechecked budget-exhausted stop does not match a reason-prefix rule and falls to the tackler.

### [MEDIUM] The verify-retry bypass is not limited to one goto

**Location:** D2 Bound.

**Issue:** The failure record stands until removed, and the driver's one-retry limit (`VerifyRetrySpent`) is not part of D2's seam, so any number of gotos onto that row from any caller pass.

### [MEDIUM] Merge-Review amendments let merge-ins through unbounded

**Location:** D5 Merge-Review bullet, D6, D7.

**Issue:** Each new merge-in is a changed identity and becomes an amendment, so it gets only the allowance; with 0, set through `run_settings`, every later merge-in is approved unreviewed, and no Decision states that.

### [MEDIUM] The pause label names no operator or orch

**Location:** D4.

**Issue:** Goto's `--by` is hidden and admits only `shed` (`internal/shedverbs/goto.go:64-66,102-103`), and loom refuses a bare pause through its `BarePause` hook, so "a `--by` label as goto does" never names a human or the orch, and D4 states neither the label's values nor its bound.

### [LOW] Relaunch's unchanged write would clear pause_reason

**Location:** D4.

**Issue:** `Relaunch` with no target or an in-effect target writes a possibly paused status unchanged (`internal/shedengine/goto.go:150-215`), and D4's clear rule would then leave paused with no reason.

### [LOW] Verdict validation does not see D2's seam

**Location:** D2.

**Issue:** The tackler and the arbiter check a verdict's goto with `CheckGotoTarget` before applying it, so a verdict naming a settled review row validates and is refused only at `Relaunch`.

### [LOW] friction_timeout_min's fate is open

**Location:** D8.

**Issue:** The key still bounds the Friction-Reflect row (`internal/loomcli/run.go:31,73`), and `bootstrap.go` names it only in comments, so "retired" leaves the key, the arm and the comments undecided.

### [LOW] D9's trigger set disagrees with its criterion

**Location:** D9, Goal, acceptance criterion.

**Issue:** The sentinel covers a surplus and a nil audit too (`internal/burlerengine/cluster.go:149-156`), while the title and criterion say fewer forks; the resumed-reviewer path is outside the attempt loop.

### [LOW] D3 overstates what alone grants a budget

**Location:** D3.

**Issue:** "Nothing else does" contradicts D1's kept enders, a reroute into the segment and a Done by a segment row.

### [LOW] D3 misstates the reroute cap's handling

**Location:** D3, Notes for the plan writer.

**Issue:** `episodeStuckCount` skips the budget-exempt `reroute_capped` entry and does not stop at it; only `rerouteCount` stops there (`internal/shedengine/reroute.go:13-21`).

### [LOW] The PR-Gate reject criterion omits the exempt skip

**Location:** acceptance criterion after a PR-Gate reject.

**Issue:** D6 says an exempt rework skips review entirely, but the criterion requires an amendment generation on every reject.

### [LOW] Carry-over idempotence is not required

**Location:** D5 last paragraph.

**Issue:** Case 1 re-runs carry-over, whose seam commits the decision record, yet only Approve and Commit are required to be idempotent.

### [NIT] D5's read order disagrees with D6

**Location:** D5, D6.

**Issue:** "Before anything else" contradicts the skip seam running first, which D6 and today's `Call` (`internal/shedadapters/bouncer.go:333-343`) both rely on.

## Rejected

- Existing-paths: the `atHardCap` doc's "a goto resets that count" goes stale under D1. Not a finding: the Scope already puts every changed package's doc in the same commit.
- Interactions: D3 against D5 cases 2 and 3 reaching the budget halt. The record makes no claim beyond case 1, so this is not a defect.
