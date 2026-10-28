# Fixer report — Plan round 2

## Changes

- F1 (MEDIUM, design), fixed as proposed, with the test assertion restated after the self-check: `_lym/plan/07-chase-clock.md`, Journal and fold.
  The `applyPosted` bullet now also states that `applyCarried` leaves `programs` alone for a carried `posted` snapshot whose `From`, read with `agentname.ParseProgram`, is `ChaserProgram`, exactly as `applyPosted` does.
  `TestJournal_Retention` gains a row that carries an unread chase and an unread sender notice past retention and refolds the journal.
  It asserts that the refolded pool's `programs` holds no `<shortname>:CREEL` entry, which is the condition on which `checkReceiverLocked` refuses a post to `CREEL`, and that the carried messages stay with their holders.
  The review's wording, "a post to CREEL is still refused", was not kept: `TestJournal_Retention` folds a bare `pool` and makes no post (`internal/creelengine/journal_test.go:412`).

## Parent decision applied

F2 is a `decision`-class finding and goes to the parent.
The parent's recorded decision, "A chaser's first tick charges (review F2, Plan round 2)", came in by creel during this round, and this round carries it out:
- In card 7's clock paragraph, `chaser.run` probes the holder once when it starts and counts that reading as the previous tick's, so the first tick charges.
  Chase 1 lands at the 15th tick after the start, and a step after a restart lands at the 15th tick after the restart.
- In card 7's `TestScheduler_Chases` rows, chase 1 is at the 15th tick, chase 2 at the 30th and `unanswered` at the 45th.
  The restart row takes its step at the 15th tick after the restart and the next step at the 30th.
- In `_lym/plan/00-overview.md`, the Shared Decision `chase-clock-cadence` states the start probe and the same tick counts.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-affected-set: no findings.
- fix-proposal, MEDIUM: the new `TestJournal_Retention` row asserted a post refusal that a pool-level test cannot make.
  Fixed as proposed: the row now asserts that the refolded `programs` holds no `CREEL` entry.
- fix-proposal, unrated note outside its lens: a carried unread chase snapshot keeps neither `Chased` nor `ChaseOf`, and card 8's reply to a chase depends on `ChaseOf`.
  Not judged as a fork finding: the rubric scopes the self-check to the change set, and this gap is in the earlier `carryRecords` design, which the fix did not change.
  It is noted here so the next review round can weigh it.

## Disputed

None.

## Deferred

None.
