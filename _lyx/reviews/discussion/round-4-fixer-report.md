# Fixer report — discussion, round 4

Every finding's premise held against the record and the code; all five are fixed in `_lyx/discussion/decision-record.md`, with a round-4 entry and four ledger picks in `_lyx/discussion/support-log.md`.

## Changes

- F1 (MEDIUM, design): fixed as proposed, refined by the self-check.
  D4 measures the budget-0 Merge-In cap from the entry's fixer baseline, the HEAD after the merge and the conflict session have committed and before the fixer's first turn, written into the entry's attempt record at the first spawn, so the cap judges only the fixer's commits and an error-stop resume measures from the same HEAD.
  Only a resume after the over-cap halt the operator reviewed writes the baseline afresh; every other resume keeps it.
  D10's pre-fix HEAD wording is unchanged.
  The budget-0 Merge-In criterion gains a large merge with a small fixer change that does not halt, and the error-stop resume.
- F2 (MEDIUM, consistency): fixed as proposed, with a split way forward.
  D11 states that `loomRerouteCapWayForward` (`internal/loomcli/wiring.go:1141`), told as `Shed.RerouteCapWayForward`, gains Finalize's fixer causes, with a test that reaches the clause.
  Not made as proposed: one way forward, "fix the failing tests", for both causes.
  An over-cap change can pass the gate and would land unreviewed on that resume, so the over-cap cause says to review the fixer's commits and fix on the task branch, as D4's and D10's other over-cap halts say.
  D13's orch rule names a parent that keeps moving or a fixer that keeps failing to clear the gate within its caps, read from the halt's reason; the fourth-reroute criterion names both fixer causes.
- F3 (LOW, consistency): fixed as proposed.
  D13 states that the conflict session's resolution, at a darn Finalize and at a loom Merge-In, is no fixer change and lands at a perch budget of 0 with the landing gate alone; Open risks gains the same line.
- F4 (LOW, consistency): fixed differently.
  D10 names the three causes, an owner pause, another machine's ownership and an operator's `lyx batten lock release`, and says the re-check stands behind every release the idle paths do not guard.
  Not made as proposed: a way forward that adds "once the run is this machine's" only for the owner and ownership causes.
  Finalize cannot tell the causes apart, because an owner pause is a live pause record (`internal/battenshed/innerrun_parentlock.go:40`), which step 1's owner checks (`internal/landingshed/finalize_owner.go`) do not read.
  The halt therefore names all three causes, with one way forward: `lyx loom resume` once no owner pause stands and the run is this machine's, which is true already after an operator's release.
  A premature resume lands nothing.
  The released-lock criterion gains the operator-release case.
- F5 (LOW, decision): fixed as proposed.
  D4's attempt record, with the baseline and the halt note, belongs to one row entry and names that entry.
  A resume continues the entry.
  A re-entry from another row starts at zero spent attempts, both for Merge-In after a Finalize reroute and for Finalize entered again after its own reroute.
  The record is cleared at Done and at that re-entry.
  A criterion covers the reset after a Finalize reroute and the kept count on a resume.

## Self-check

- fix-claims: no findings.
- fix-consistency, LOW, the released-lock criterion has no operator-release case: fixed, added.
- fix-consistency, LOW, D4's "D10's pre-fix HEAD is the same measure at Finalize" does not hold across Finalize's resume and retry pass: fixed, the sentence is removed.
- fix-consistency, LOW, the new picks have no ledger lines: fixed, four lines added to the question ledger.
- fix-proposal, MEDIUM, and fix-way-forward, MEDIUM (merged), the reroute-cap clause's "fix the failing tests" lets an unreviewed over-cap change land on resume: fixed, the over-cap cause says to review the fixer's commits and fix, in D11 and the criterion.
- fix-proposal, MEDIUM, and fix-way-forward, MEDIUM (merged), rewriting the baseline after any landing-stop resume lets fixer commits made before an `escalate` escape the cap, and the row cannot tell a landing-stop resume from an error-stop one: fixed, only a resume after the over-cap halt, which is noted in the attempt record, rewrites the baseline.
- fix-proposal, LOW, D13's "bounded only by" sentence contradicts the conflict-session sentence that follows it: fixed, the bound sentence is reworded so the caps bound fixer changes alone.
- fix-way-forward, MEDIUM, a resumed Merge-In after a spent cap must spawn a fixer it has no attempts for: fixed, the spent-cap halt is noted in the attempt record, and the resumed Merge-In reads the operator's fix as the alignment, advancing over a passing gate and halting again over a failing one.
- fix-way-forward, LOW, the re-check halt's precondition gives the orch no way to check it: fixed differently, D10 states that a premature resume lands nothing, since step 1's owner check stops an owned-elsewhere run and the re-check catches a lock released again by the owner-pause arm.
  No lyx command shows a live owner pause, so the halt cannot name one.

## Disputed

None.

## Deferred

None.
