# Fixer report — discussion round 1

## Changes

- F1 (fixed as proposed): D5.4 gains a Bound stating what a declaration lets through and what bounds it; D5.2's Bound names the `--by parent|operator|arbiter` caller guards; new D5.7 has the Webster-Review rubric check the branch's `PATTERN.md` and `pattern/` diff lands each declared entry, a miss being BLOCKING; Scope In and an Acceptance criterion match.
- F2 (fixed as proposed): D2.8 now has the verify gate closure collect `fix:` commits (`fix_commits`) and `Pattern-Departure:` trailers (`departures`) on its existing `fixCommitRejection` walk and hand them to the Master in the gate result; the master stencil's exception is dropped; D2.4, Scope In, Auto-mode assumptions and the Acceptance criteria match; the reply-as-exception option moved to support-log.md Rejected alternatives with the operator ruling (board entry: "The Master still never reads raw fork output; the departure reaches it through Go") as the reason, replacing the Go-side-list rejection; question ledger Q5 revised.
- F3 (fixed as proposed): D2.6's Bound names the plan's `## verify:` command; the `OK`-on-uncaused-failure rule applies only when the plan declares that section, otherwise `FAILED` as today; the fork learns the case from a required `plan_verify` marker rendered from the plan into `webster-body-implementer.md`; Acceptance criterion added for both cases.
- F4 (fixed differently in one detail): D5.4's way forward is now the parent or operator fixing the named record section, confirming with `lyx loom validate-discussion`, then re-stepping the row; each caller's row is placed: websterengine in `## webster`, loomengine in `## loom`, the Bouncer in `## shed` beside its existing review-segment rows. The fix proposed placing burler's row under the loom row whose segment spawns it; `refusal-spec.md` keeps review-segment rows (Bouncer, lines 196-203) under `## shed`, not `## loom`, so burlerengine propagates its error to the spawning Burler-round row, whose row goes in `## shed`. Question ledger Q13 revised.
- F5 (fixed as proposed): D1.4's second half reads "no other role and no other card runs a script or tool that rewrites file content beyond what this directive already allows".
- F6 (fixed as proposed): D5.1 states the PATTERN edit itself lands in a Webster card, never during Discussion or Plan; the Scope Out bullet is cut to its pointer; the "D5 answers the same need" clause moved to support-log.md Rejected alternatives.

The support log's `## Review rounds` records round 1. `lyx loom validate-discussion` passes.

## Disputed

None.

## Deferred

None.
