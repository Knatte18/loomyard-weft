```yaml
lens: bound-breaker
outcome: reported
```

**Claims attacked:**

- D3 "reuses a stored ruling while the file's diff is unchanged" vs the way forward "a trailer commit, then `lym loom resume`" — **broken**.
- D4 "PR-Review again … with `pr_review_ask_parent` on, asks the parent again" — **broken**.
- D4 fix-here fallback "when … the edge's target row's entry gate does not hold" as the bound on rerouting into a perch that cannot review — **broken** (darn, Code-Review budget 0).
- D2 bound "D4 keeps PR-Review's fixes small" — **broken** (repeated small sets; agent can skip the verb).
- D1/D2/D3 trailer "measured cause" as the bound on clearing a loosened check — **broken** (unverified; the fixer can write it).
- D4 "The verb refuses while the worktree holds uncommitted changes … so the row's wake meets a clean tree" — **broken** (nothing holds the tree clean between the verb's answer and the wake).
- D4 acceptance "re-entered PR-Review row opens a new review session" — **held weakly**; existing code resumes the recorded session.
- D4 "reroute edge spends no bounce budget, fresh episode, capped" — held (`internal/shedengine/reroute.go:43-68`, `doc.go:112-116`).
- D4 reroute cap bounding the send-back — held; `rerouteCount` resets only on a PR-Review Done, which never comes between send-backs (`reroute.go:16-31`).
- D4 "returns a Stuck verdict with `Reroute` set" — held as long as Halt stays false (`run.go:430`; today `stuckVerdict(..., true)` always halts, `prreview.go:117`).
- D4 "the row's existing stop ends the session" — held (`prsession.go:249-256`, `prreview.go:83`).
- D4 PR-findings input on an unchanged subject starts a generation — held if the input seam reports the send-back file newer than `SettledAt` (`bouncer.go:729-736`).
- D4 send-back mark wakes the wait — held, `RecordChanges` ends the pause within `prWaitFloor` (`prreview.go:548-549`, `582-598`).
- D6 "a path of its own per send-back … never overwrites" — held via `NotePath` (`friction.go:120-160`) with a distinct stem from the agent's.
- D3 "review base = last merged parent tip, so Parent-Align changes are no subject" — held (rubric `loom-rubric-code-review.md:35-40`); repo code, not a tool fact.
- D3 "both diff modes" — held (lines and command, `verify.go:61-69`).
- D5 run_settings override — held (`landingdeps.go:446`).

**Findings:**

1. **BLOCKING · design** — `_lym/discussion/decision-record.md` D3 (lines 67, 70-71).
   Summary: A trailer commit never clears an `uncaused` ruling: the ruling is "keyed by a hash of the file's diff, and reuse[d] … while that diff is unchanged", and the way forward is an (empty) commit carrying a `Loosened-Check:` trailer, which leaves the file's diff unchanged. On `lym loom resume`, Drift-Verify reuses the stored `uncaused` ruling and goes Stuck again, no judge spawned. The only exit is a revert. No acceptance criterion covers it (line 158 only checks the ways forward are named, not that following one un-sticks).
   Fix: key the ruling on diff hash plus the range's `Loosened-Check:` trailers (or never reuse an `uncaused` ruling); add a criterion that a trailer commit then resume rules the file `caused`.
   Evidence: decision-record.md:67, 71, 158; reuse path `internal/driftengine/verify.go:246, 335-343`.

2. **BLOCKING · design** — `_lym/discussion/decision-record.md` D4 fix-here fallback (line 95) with D4 bound (lines 108-109).
   Summary: In darn with Code-Review budget 0, the perch is off but Code-Bouncer carries no entry gate — `zero_rounds` makes it settle approved at once. The verb's "target row's entry gate holds" check passes, so it sends back; Code-Bouncer approves without reading the findings; Drift-Recheck; Describe is skipped (`budget.Code-Review` gate); PR-Review re-entered with the same findings; repeat until the reroute cap halts the run. The findings are never reviewed or fixed.
   Fix: the verb answers fix-here when the target perch is off by any means (budget 0 / `zero_rounds`), not only an entry gate; add a criterion.
   Evidence: `internal/loomrecipe/doc.go:88-92`; `contracts/recipes/darn-recipe.yaml:160-163, 270`; `internal/shedadapters/bouncer.go:396-398`.

3. **MEDIUM · design** — `_lym/discussion/decision-record.md` D4 (line 103) and its acceptance (line 166).
   Summary: The claim that the re-entered PR-Review "asks the parent again" is false against current code: `askParentReview` returns early when `rec.ParentReviewAsked` is set, which the first ask sets, and `recordPullRequest` on a found record never clears it. D4 states no reset. After a send-back the second PR-Review waits on the open PR with no review request to the parent.
   Fix: D4 states that the send-back mark write (or its clearing) resets `parent_review_asked`; add a criterion.
   Evidence: `internal/landingshed/prreview.go:343-367`, `445-455`; `internal/pullrequest/record.go:44-45`.

4. **MEDIUM · design** — `_lym/discussion/decision-record.md` D2 bound (lines 46-47) and D4 (lines 91-95, 105-106).
   Summary: "D4 keeps PR-Review's fixes small" holds per findings set, not per run. Counterexample: parent sends 3 findings → agent fixes in place, asks again → parent sends 3 more → fix-here again; any number of sub-threshold sets passes. The agent can also skip the verb, or write the findings file itself with severities/count of its choice, since it transcribes the parent's creel answer. All those commits land after Drift-Recheck, unchecked by D1/D3. The bound states neither the cumulative nor the skip path.
   Fix: state both as accepted with what limits them, or bound the cumulative count (e.g. count fix-here findings across the PR-Review episode, or have Go, not the agent, hand the parent's findings to the verb).
   Evidence: decision-record.md:46-47, 87-95; `prreview.go:372-375` (the parent's answer goes to the agent, "which the row never reads").

5. **MEDIUM · design** — `_lym/discussion/decision-record.md` D1 (line 27), D2 (lines 40-47), D3 (line 64).
   Summary: A `Loosened-Check:` trailer clears both the D1 finding and the D3 `uncaused` ruling on its text alone; no one verifies the "measured cause" was measured. Counterexample: Code-Burler-Fix, told to fix a D1 "loosened check" finding, adds an empty commit with an invented figure, and both checks pass. D2's only stated bound is the post-Drift-Recheck window; the record does not say what an unverified trailer lets through or what bounds it (the one partial bound — D3 surfaces the cause in the regression check section, so the parent sees it at PR-Review — is never named as one).
   Fix: state the bound in D2 (cause is attested, not verified; surfaced to the parent in the PR body), or forbid the fixer from authoring the trailer.
   Evidence: decision-record.md:27-28, 42, 64, 72.

6. **LOW · design** — `_lym/discussion/decision-record.md` D4 (lines 92-97).
   Summary: After the verb answers "end your turn", the agent can still edit before the row's wake and stop (the stop runs only in `Call`'s `endSession`, after the verdict), so Code-Bouncer can start on a dirty worktree; the verb's refusal is a point-in-time check.
   Fix: the row's send-back wake checks the tree clean (as `sessionStopWanted` does for continue) before rerouting.
   Evidence: `prreview.go:657-669`, `686-695`; `prsession.go:249-256`.

7. **LOW · consistency** — `_lym/discussion/decision-record.md` acceptance line 166 vs D4.
   Summary: The criterion says the re-entered row "opens a new review session", but `ensureSession` resumes the recorded `SessionID`, so the re-entered agent continues the conversation in which it was told to end its turn; D4 states no mechanism for a new session.
   Fix: D4 says whether the send-back clears the recorded session id, or the criterion is reworded to "resumes the session with a new review request".
   Evidence: `internal/landingshed/prsession.go:65-88`.

Tool calls:
1. `prsession.go` ensureSession/endSession — manifest lacked how a re-entered row starts and when it stops its session.
2. `FullGeneration` and the Bouncer Call re-entry — manifest lacked whether a review input on an unchanged subject starts a generation.
3. `budget.Code-Review` gate and darn Parent-Align — needed to test the fix-here entry-gate bound.
4. `loomrecipe/doc.go` budget-0 perch and the skip seam — needed to confirm an off perch settles approved with no entry gate.
