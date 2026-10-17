# Round 2 fixer report — discussion

## Changes

- **F1** (MEDIUM, design): fixed as proposed, then widened at self-check.
  D7 now requires presence agreement within each review segment, beside reference agreement.
  The BurlerFix row names `fixer_fan_setting` exactly when its BurlerReview row names it or sets a profile `fix-fan`.
  The BurlerReview row names `reviewer_fan_setting` exactly when the Bouncer does.
  D7's rationale now says the wiring gave reference and presence agreement by construction, and that the test keeps the unrecorded-round refusal from being bypassed by omission.
  D5 gains the sentence after line 91: a BurlerFix row naming no `fixer_fan_setting` reads no recorded fans and runs the single fixer, bounded by D7's presence agreement.
  The acceptance scenario on segment fan settings now also covers presence.
- **F2** (LOW, consistency): fixed as proposed.
  D8's refusal sentence now gives separate ways forward: `Routing` points at the loom wiring's segment-to-perch table, and `DarnRouting` at the `Darn-Review` budget entry the darn wiring builds from `darn.yaml`'s `review_max_bounces`.
  The acceptance criterion on `Routing` and `DarnRouting` refusals, which named only the perch table, was aligned to the same split.
- **F3** (NIT, consistency): fixed as proposed.
  D8's negative-budget refusal now takes the key from `reviewbudget.Caps` and the file from the segment's module (`loom.yaml` or `darn.yaml`), as D1's `SegmentSettings` carries them.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-way-forward: no findings.
- fix-proposal, MEDIUM: under F1's presence rule as proposed, a review row with a profile `fix-fan` and no `fixer_fan_setting` passed D7 beside a fix row naming no key, so the fix row dropped the recorded fan, contradicting the new D5 bound (`internal/shedrecipe/entries_burler.go:113` records a round from a profile `fix-fan` alone; `internal/shedrecipe/entries_burler.go:184` requires the record only when the fix row has a fan key).
  Disposition: fixed with my own change.
  D7's presence rule and its acceptance scenario now count a profile `fix-fan` as a route that obliges the fix row to name `fixer_fan_setting`.

## Disputed

None.

## Deferred

None.
