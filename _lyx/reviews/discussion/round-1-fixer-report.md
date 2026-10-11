# Fixer report — discussion round 1

## Changes

- F1 (BLOCKING), fixed differently.
  D11's Bound now lets auto-accept fire only on a harvest, a verdict the judge wrote in this Bouncer call over the round's fixer report; a replay of an earlier round's recorded CONTINUE escalates as today.
  Premise confirmed: `doneExit` in `internal/shedadapters/burler.go` archives only the review and fixer report on a gate failure, so `ResolveRound` falls back to the prior judged round and `Call` in `internal/shedadapters/bouncer.go` replays it through `settle` → `settleUnconverged` → `settleUndecided` with `spawned` false, where `budgetReached` would trigger the accept.
  Not made: the proposed bound "the highest round with a review on disk and no later archived gate-failed round".
  `archiveStaleOutputs` renames files to stamped siblings with no record of why, so the Bouncer has no way to tell a gate-failed archive from an orphan one without new state; the `spawned` flag `settle` already carries separates a harvest from a replay with none.
  D10 now states that under a budget of 1, a fix gate exhausted in round 1 meets Shed's generic budget block and halts for a human.
  Added the proposed acceptance criterion for the gate-exhausted-after-spent-budget escalation.
- F2 (MEDIUM), fixed as proposed.
  D3 states the acceptance record is one of the round's paths, moved with the review and fixer report by every archive of the round.
  D4's "review changed after acceptance" bullet archives all three so a resume re-reviews.
  The criterion "A review edited after acceptance fails the fix row" now also states the archive and the re-review, replacing the shorter line rather than adding a second one.
- F3 (MEDIUM), fixed as proposed, using the first of the two proposed options.
  D7 states the current round's review gate refuses a review whose prose `### [SEVERITY]` finding blocks do not match its `findings` entries one for one; D4's skip rests on it.
  Premise confirmed: `reviewHeader` in `internal/burlerengine/verdict.go` decodes without known-field checks.
  The alternative option, refusing unknown top-level keys, was not taken: that comment records the tolerant decode as a deliberate choice, "agent-written metadata in the header is harmless noise", and the one-for-one match catches the misnamed key without reversing it.
  Added the proposed acceptance criterion.

## Disputed

None.

## Deferred

None.
