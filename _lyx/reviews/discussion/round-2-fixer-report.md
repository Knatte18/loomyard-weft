# Fixer report — discussion round 2

## Changes

All edits are in `_lyx/discussion/decision-record.md`.

- F1 (BLOCKING), fixed as proposed.
  D2 opens the rule's window at whichever comes first, the soft cap by the bounce-budget measure or a round numbered at or past the hard cap, and replaces "Rounds below it run as today" with "Rounds before the window opens run as today", naming the goto that resets Shed's count while round numbers carry on.
  D3 states the hard cap is enforced whatever Shed's count reads, naming `lyx loom goto`.
  New acceptance criterion: after a goto resets the bounce count, a `CONTINUE` round at the hard cap escalates `hard-cap` with a design-wrong entry open and is accepted with none.
- F2 (MEDIUM), fixed as proposed.
  D3's load refusals and D5's verb refusal name the layer the resolved key comes from: `lyx board upsert` then `lyx batten run <slug>` for a task's `run_settings` value, `lyx config loom|darn --set …` from the prime for a hub value.
  New acceptance criterion for the `run_settings` route on the `circling continue` refusal.
- F3 (MEDIUM), fixed as proposed.
  D2 states that on a recorded continue at or past the hard cap the Bouncer moves both the continue decision file and the existing escalation record aside under retired names, then writes a fresh `hard-cap` record and brief; "keeps it" is scoped to rounds without a recorded continue at the hard cap; the replay sentence covers both moves.
  The matching acceptance criterion now names the moved record and the fresh `hard-cap` record and brief.
- F4 (MEDIUM), fixed as proposed.
  D2's Bound states the carry-over bound for loom's perches only, into the decision record's `## Open risks`, and names the Darn-Review bound: the soft-cap window, the `automatic` flag and the operator's review at PR-Gate.
  D2 step 2's accept path now says carry-over happens where the recipe sets `carry_over`, to stay consistent with the Bound.
- F5 (LOW), fixed as proposed.
  The earned-`CIRCLING` criterion is qualified to a perch without auto-accept, a replay, or a `blocking_escalates` round with an open BLOCKING entry.
- F6 (LOW), fixed as proposed.
  D2 step 4 says earned `CIRCLING` covers an earlier-round key from `review_circling_checkpoint` on, and that below it a key returning after a skipped round runs on until the hard cap stops it.

## Disputed

None.

## Deferred

None.
