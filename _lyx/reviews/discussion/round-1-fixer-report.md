# Fixer report — discussion round 1

## Changes

- F1 (fixed as proposed, second option): D8 keeps a replay's exclusion from auto-accept, and its rationale now says such a replay is settled by the rule.
  D2's sentence is narrowed: harvest and replay agree on perches without auto-accept, and the auto-accept difference points at D8.
  The replay criterion now says an unsettled replay is settled by the rule on every perch.
- F2 (fixed as proposed): D2 defines the soft cap once, as the bounce-budget seam's count at or above the budget, and says that count can reach the budget below round S because it also holds the seed, degrades and rejudges.
  D3 measures the hard cap by the round's number within the generation, says Shed's count never measures it, and rewrites the equal-caps sentence to match.
  D4 names both measures.
  "Past the soft cap" is gone from step 1 and step 4 (see F3), so it needs no measure of its own.
  Acceptance criterion added for a soft cap reached below round S after a counted degrade.
- F3 (fixed as proposed, with one change of window): step 4 now matches entries by ledger key, requires the previous round's entry to count as design-wrong too (missing tag included), and consults only the immediately previous round, with the reason.
  Instead of "past the soft cap", the check runs when the previous round's decision file records a `continue`, an on-disk condition that needs no window marker.
  Step 1's previous-ledger clause follows it.
  A4, the support log's Q4 and the acceptance criteria are updated, with a new criterion for a key that was LOW or `mechanical` in the previous round.
- F4 (fixed as proposed): D2's Bound states that a mis-tag lets an unreviewed design change into the artifact and that nothing re-reviews it.
  It is bounded by the soft-cap window, the carry-over of MEDIUM-or-worse open entries into `## Open risks` with class and severity, and the decision file's automatic flag.
  The Open risks line points at the Bound.
- F5 (fixed as proposed): D6's accept-only restriction follows the round (at or past the hard cap, whatever the cause), with a criterion for a `budget` escalation at the hard cap.
- F6 (fixed as proposed): D2 has the Bouncer move a stale `continue` decision file aside before writing the `hard-cap` escalation record, so `circling accept` can record a decision.
  It also states what a replay between the two writes does.
  The criterion now names the accept that follows.
- F7 (fixed as proposed): new D9 states the orch stencil's guidance: continue only below the hard cap, and the small-task advice keeps `review_max_bounces: 2` and adds `review_hard_max_bounces: 3`, with rationale.
  Scope points at D9, and A9, Q10 and the stencil criterion are added.
- F8 (fixed as proposed): step 5 records the rule's continue with cause `budget`, and D6 says rule-written decision files carry `budget` and the `automatic` flag.
  Criterion added.

## Disputed

None.

## Deferred

None.
