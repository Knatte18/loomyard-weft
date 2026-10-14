# Round 2 fixer report — discussion

## Changes

- F1, fixed differently: D7 now names the mechanism.
  The rubric files an unrecorded parent correction as a `decision`-class MEDIUM finding.
  Its fix text tells the fixer to leave the code and defer the finding to the parent, who records or refuses it with `lyx loom decision add`.
  D7 states that Darn-Bouncer accepts with that entry open.
  The judge's CONTINUE rules and the circling escalation fire only on gating findings, and only the `design` class is gating (`internal/shedadapters/bouncerprompt.go`: "A finding is gating when its class is `design`").
  `carriesOver` takes every open entry at MEDIUM or worse, so D8 files it under `## Open risks`.
  Not made as proposed: the review's recommendation has the fixer *dispute* the finding.
  The fixer's dispute grounds are `premise false` and `every fix worse`, and neither fits a finding whose premise holds and whose settlement belongs to another party.
  The fix instructions route that case to a deferral with a reason, so the record says "defer, naming the parent".
  Acceptance line 167 is reworded.
  Line 168 is replaced by two lines: the fixer leaves the deviating code in place, and the Bouncer accepts with the finding open and carries it over.
- F2, fixed as proposed: D9 adds the rule that a carried-over open risk naming an unrecorded parent correction is escalated to the parent at `pr-gate`, never approved or rejected, carried by the same told value.
  Added an acceptance line for that escalation.
- F3, fixed as proposed: D4's reference is now every `Added after Discussion` heading that any commit of the record in the weft's history has held, never the latest committed copy.
  D5's refusal adds the same history check.
  The verb confirms after its commit that the committed record holds its own entry, and refuses naming the entry when it does not.
  D5's Bound is restated to match.
  Added acceptance lines: a record committed through `lyx fabric commit` without an earlier entry is still refused, naming it, and the verb refuses when its entry is missing after the commit.
  Added a plan-writer note on reading the record's history through `fabricengine`.
- F4, fixed as proposed: D10 adds that the `pr-gate` scope-deviation definition comes from the told value.
  Loom's wiring keeps the recorded-decision clause, and darn's names only the board entry and its `Added after Discussion` entries.
  The darn PR-Align acceptance line covers the definition, with a loom line that keeps the clause.

`lyx loom validate-discussion` passes on the edited record.

## Disputed

None.

## Deferred

None.
