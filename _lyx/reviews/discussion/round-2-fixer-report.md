# Fixer report — discussion round 2

## Changes

- F1, fixed as proposed: D7's last bullet gives the way forward of moving the fixer's model key to an engine with forks and resuming, and says that setting the fixer fan entry to `""` reaches only rounds whose review half starts after the edit, per D2.
  Its spec-row sentence carries the same way forward.
  No Acceptance criterion named this refusal, so a new criterion states it with that way forward.
  D2's pinning is unchanged.
- F2, fixed as proposed: D7's `fix-fan` bullet hands the round's fixer fan list to the BurlerReview row beside the reviewer fan, the review half records the picked entry per D2, and the BurlerFix row reads the recorded name.
  The recipe-profile `fix-fan` key and its conflict refusal move to the BurlerReview row's profile, and `lyx burler run` reads `fix-fan` from its own profile.
- F3, fixed as proposed: D2's pinning bullet writes both names, empty ones included, before the reviewer spawn, fails the review half on a failed write, and has a fix half or Bouncer with no recorded fan names refuse with the way forward to re-step the review row, with a spec row.
  A matching Acceptance criterion is added.
- F4, fixed as proposed: D4's first bullet gains a checklist line for every test, caller, doc and fixture that reads or asserts what the target changes; D8's `fix-affected-set` reruns each finding's own search, one tool call per finding; Open risks names the 6-call cap as unbenched against the Plan lenses.
  D6's `affected-set` text now finds the set from the pre-read and a search for what it lacks, instead of searching the whole repository, so it agrees with the new checklist line and the cap.
- F5, fixed as proposed: Open risks' first bullet says the bench ran its fork rules under today's Go boilerplate and holistic paragraph, which D4 removes.
- F6, fixed as proposed: D4's same-turn sub-bullet is deleted.

## Disputed

None.

## Deferred

None.
