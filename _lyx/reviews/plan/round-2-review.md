---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/29-finalize-gate-and-fixer.md:37
    summary: A Finalize over-cap halt at a budget-0 route back cannot be lifted by its own way forward, because the resumed entry judges the same Baseline change by the caps again and halts again
    fix: "In card 29 Intent, make the over-cap halt of a route back that reaches no review note over-cap in the entry's attempt record (the Halt note card 10 defines), exactly as card 26 does at Merge-In. Add that a resumed Finalize whose record notes over-cap writes Baseline afresh at the HEAD it resumes from and clears the note in the same write, reads the operator's review as acceptance of the fixer's commits, and then runs the gate: over a passing gate it proceeds to the landing without judging the change by the caps, and over a failing gate it runs the fixer on the attempts the entry has left, or reroutes or halts as a spent cap does when none are left. A resumed entry with no over-cap note keeps Baseline as today. Add to TestFinalize_GateAndFixer a row where a two-turn fix whose last turn edits a _test.go file halts over-cap with RerouteReviewed false, and the resume over a passing gate lands instead of halting again. Update the card 29 ImpactSummary only if its wording changes. The edit is Intent and test prose only, so no target changes and no validate-plan overlay applies."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/26-merge-in-fixer.md:46
    summary: The over-cap note that makes a resume write the baseline afresh is never cleared and never overwritten by a later halt, so a later resume of the same entry rewrites the baseline again and lets earlier fixer commits escape the cap
    fix: "In card 26 Intent, state that the resume that writes Baseline afresh after an over-cap halt clears Halt in the same record write, and that every halt other than spent and over-cap leaves Halt empty, so Halt names only the halt that last stopped the entry as card 10 defines it. Add to TestMergeIn_Fixer a row under CodeReviewOff where an over-cap halt is resumed over a failing gate, the fixer turn commits a change and escalates, and the second resume calls LineChange from the baseline written at the first resume, never from the HEAD it resumes from, so the escalated turn's commits still count (D4). Edit is Intent and test prose only, so no target changes and no validate-plan overlay applies."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/07-fixer-session-table.md:29
    summary: Card 7 makes AlignedTip an optional value empty on a run's first Merge-In, but card 26 reads that case from the entry's AlignFrom fork point, and card 11 never says what its tips argument carries, so the fixer stencil can reach the first Merge-In with no start tip
    fix: "In card 7 Intent, define FixerInputs.AlignedTip as the tip the parent's changes are read from, the merge record's aligned tip or, when absent, the entry's AlignFrom fork point, and keep it optional only for finalize mode and the darn run; adjust the sentence that lists AlignedTip as empty on a run's first Merge-In and the TestFixerTable row that renders an empty optional AlignedTip to use finalize mode. In card 11 Intent, say that the tips argument of fixerRun.run is the from tip and the merged tip, and that run fills FixerInputs.AlignedTip and MergedTip from it. In card 26 Intent, say that align passes the aligned tip, or AlignFrom when it is absent, as the from tip. Add to TestMergeIn_Fixer's first-Merge-In row an assertion that the table the fixer is spawned with carries the fork point as its aligned tip. All three edits are Intent and test prose, so no target changes and no validate-plan overlay applies."
---

### [BLOCKING] A Finalize over-cap halt at a budget-0 route back cannot be lifted by its own way forward

**Location:** `_lyx/plan/29-finalize-gate-and-fixer.md:37`

**Issue:**
Card 29 halts with `LandingStopPrefix` where the route back reaches no review, on a spent cap or an over-cap change, with the way forward to review and fix on the task branch, then `lyx loom resume` (D10 step 4).
The card keeps `Baseline` across every resume of the entry and defines a resumed entry with no attempts left as: over a passing gate, judge `LineChange(Baseline, HEAD)` by the caps and halt an over-cap change again.
An over-cap halt on the last attempt leaves `Spent` at `finalize_fix_attempts`: the fixer's second turn makes the gate pass but edits a test file, or two turns of 30 lines sum past 40.
The operator reviews, resumes, and the entry has no attempts left, the gate passes, and the same `Baseline..HEAD` change is judged over the cap and halts again with the same reason.
Nothing in the card writes `Baseline` afresh or marks the halt, so the halt the card prescribes for the budget-0 recipes (darn at `review_max_bounces: 0`, loom at `code_max_bounces: 0`) has no resume that lands.
Card 26 has the mechanism at Merge-In (D4: the resume after the over-cap halt writes the baseline afresh and reads the operator's review as the alignment), and D10 says only that the halt names that way forward, so Finalize needs the same state; card 10's `Halt` note already allows `over-cap`.
Where the route back reaches a review the same case reroutes and converges, so only the no-review route is affected.
No test of card 29 resumes an over-cap halt, which is why the loop is not caught.
The prior round's fixer report records adding the caps judgement over a passing gate to the exhausted-entry resume, which closed the unjudged landing and opened this loop; no prior finding was disputed or deferred, so none is re-raised.

### [MEDIUM] The over-cap note is never cleared, so a later resume rewrites the baseline again

**Location:** `_lyx/plan/26-merge-in-fixer.md:46`

**Issue:**
D4 says only a resume after the over-cap halt writes the baseline afresh, and every other resume keeps it, so fixer commits made before an `escalate` or another halt still count.
Card 26 notes `over-cap` in the record on that halt and writes the baseline afresh on the resume after it, but states neither that the refresh clears the note nor that a later halt of another kind overwrites it.
Card 10 defines `Halt` as the note of the halt that last stopped the entry, yet card 26 writes a note only for `spent` and `over-cap`.
Over a failing gate after the over-cap resume the fixer runs; if its turn commits a change and escalates, the entry halts with `Halt` still reading `over-cap`, and the next resume writes the baseline afresh at a HEAD that already holds the escalated turn's commits, which no longer count against the caps.
At a Code-Review budget of 0 this lets an unreviewed fixer change past the caps land, the case D4 and D13 bound by the caps alone.

### [MEDIUM] AlignedTip is empty on the first Merge-In, where the fixer most needs the fork point

**Location:** `_lyx/plan/07-fixer-session-table.md:29`

**Issue:**
Card 7's `merge-in` stencil reads what the parent brought in since `{{.aligned_tip}}` up to `{{.merged_tip}}`, and card 7 lists `AlignedTip` among the optional values, "empty ... on a run's first Merge-In".
Card 26 spawns the fixer on a first Merge-In over a parent that moved since the fork and reads that range from the entry's `AlignFrom`, never from an empty tip.
Card 11's `fixerRun.run` takes `tips [2]string` and no card says which two values they are or how they reach `FixerInputs`, so as written an implementer following card 7 renders an empty start tip on exactly the run the fork-point rule serves, and the session cannot tell what to read.
The three cards need one rule for the fixer's start tip.

## Rejected

- Focus 1, the point where `AlignFrom` is recorded: not a defect.
  Card 26 writes `AlignFrom` into the entry's attempt record right after `readBefore`, before the merge, and every later `Call` and `align` of the entry read it back, so an error stop after the merge and the `landing` gate and before `align` resumes over the parent commits the first merge brought in.
  The test row for a first Merge-In that stops with an error after the merge pins it, and the attempt record is cleared only at a Done, after the aligned tip is written.
- Focus 2, `Spent` at the cap with no `spent` or over-cap `Halt` note: not a defect at Merge-In, a defect at Finalize through F1.
  Card 10 keys exhaustion on `Spent` alone, card 11 charges each turn when it starts and takes back only an `escalate` or a failed spawn, and cards 26 and 29 take the exhausted-entry path before any spawn, so no resume spawns a turn past `merge_fix_attempts` or `finalize_fix_attempts`.
  What a resumed exhausted entry reads over a passing gate is what F1 and F2 settle.
- Focus 3, the hand-back mark of every halt that grants a fresh cap: not a defect.
  The undecodable-attempt-record halt of cards 26 and 29 opens with `LandingStopPrefix`, `landingshed.HandsBack` (card 34) answers true for it, and cards 35, 36 and 37 read that one predicate for the Go driver, the arbiter and batten's re-step.
  The LLM driver reaches a blocked stop through the arbiter, which card 36 refuses, and card 38 closes the `binary_changed` sites, so only the orch's `lyx loom resume` lifts it.
  No other halt of the plan's way forward grants a fresh cap.
