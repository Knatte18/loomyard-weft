# Fixer report — discussion round 3

## Changes

- F1 (fixed as proposed, first alternative): D2 now has the Discussion or Plan fix half archive the round's review, fixer report and acceptance record together before refusing, as the fix producer's changed-review exit does, so the `goto` to the review row finds no accepted review, spawns a fresh reviewer and writes the names before the spawn.
  New acceptance criterion: re-stepping a round whose accepted review has a name-less record ends with the names recorded.
- F2 (fixed as proposed): D2 names the presence marker `fans_recorded: true`, written with `fan` and a new `fix_fan`, and says why the flag decides and not `fan` (plain string, no `omitempty`, so an old binary's solo record decodes as `""`).
  It states that no later merge sets or clears the three keys, that a record `mergeRoundUsage` replaced starts without the flag, and that an undecodable record takes the same refusal; one acceptance criterion row for the decode-failure case.
- F3 (fixed as proposed): D2 puts the pre-spawn write on the spawn path only, after the live-reviewer probe finds none; a resumed live reviewer keeps its names and the hand-on path writes nothing; the acceptance-time merge writes only the lenses and the review half.
  Both added to the fan-list-changed criterion.
- F4 (fixed as proposed, degrade): the refusal is scoped to the Discussion and Plan fix halves; the Webster, Merge and Darn review halves record no names and their fix halves run the single fixer.
  The Bouncer's one read, the judged round's fan for `exclude_lenses`, degrades: a missing, flag-less or undecodable record counts as no fan, the facts file notes it as `computeLensFacts` does today, and `lensesRunUnderFan` keeps treating it as ineligible.
  Acceptance criteria: the old fix-half-or-Bouncer row is split into the fix-half refusal row, a Webster/Merge/Darn row and a Bouncer-degrades row.
- F5 (fixed as proposed): D7 adds `RunOpts.FixKey` and a told per-row key pair (fixer model key, fixer fan key) for each Discussion and Plan BurlerFix row beside the per-row model lists.
  For `lyx burler run` the refusal names the `shuttle.yaml` default engine and the profile's `fix-fan`, with its own way forward, and the spec row quotes one wording with the two key names as slots.
  The engine-refusal criterion covers the standalone wording.
- F6 (fixed as proposed): D8 `fix-affected-set` reruns the most severe findings' searches first and lists each unchecked finding for the fixer-report's `## Self-check`.
  D7 states the fixer fork cap (6 further tool calls) where it adopts the self-check block, and D4 states the reviewer fork cap (6 tool calls) it carries over from the bench's fork rules, since Open risks, now naming both caps by decision, no longer holds the number.
- Consistency follow-ons: D9 and its usage-record criterion now place the fixer's lenses and fork count beside the `fix_fan` D2 records, and a single round records an empty `fix_fan`; D2 and D7 say "rounds whose reviewer is spawned after the edit" in place of "whose review half starts".
- Support log: a Round 3 entry under Review rounds.

## Disputed

None.

## Deferred

None.
