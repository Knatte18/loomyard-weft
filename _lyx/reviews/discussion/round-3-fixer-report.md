# Fixer report — Discussion-Review round 3

## Changes

- F1 (MEDIUM, design), fixed as proposed: D9's first bullet gains a Bound line stating that a fasit-contradicting or rubric-excluded finding can now be applied, bounded by the judge's CONTINUE and a later round reverting it, and unbounded when that CONTINUE is auto-accepted at a spent budget.
  `## Open risks` gains a matching line: such an applied change is committed under auto-accept and is not carried over, since it is not an open finding.
- F2 (MEDIUM, consistency), fixed as proposed: `## Acceptance criteria` gains three criteria (an existing escalation record stands over auto-accept; a recorded parent decision settles as recorded whatever the flag; a restart between the judge's CONTINUE and settling escalates on resume), and the auto-accept criterion now reads "neither an escalation record nor a parent notice is written".
- F3 (LOW, consistency), fixed as proposed: premise confirmed against `BurlerProducer.Call` in `internal/shedadapters/burler.go`, whose attempt loop retries a died or timed-out first attempt and errors only on two consecutive died/timeout outcomes, or on an unrecognized outcome.
  D3's reviewer exit, D4's fixer-failure bullet and the restart criterion now read "two consecutive attempts died or timed out, or an unrecognized outcome".
- F4 (LOW, scope), fixed as proposed: premise confirmed, `BurlerProducer.Call` returns an `ErrHalfNotStopped` run error bare, before `failureExit`, so nothing is archived and nothing retried.
  D3 gains the reviewer's not-stopped exit and D4 gains a fixer not-stopped bullet, each with nothing archived and no retry.
- F5 (LOW, consistency), fixed as proposed: D6 now says that on a zero-findings round whose gates failed, the fixer's prompt also holds the gate's findings and the target paths they name.

`lyx loom validate-discussion` passes on the edited pair.

## Disputed

None.

## Deferred

None.
