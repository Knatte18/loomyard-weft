# Round 1 fixer-report — discussion

All changes are in `_lyx/discussion/decision-record.md`; `support-log.md` is unchanged.

## Changes

- **F1 (MEDIUM)** — fixed as proposed.
  D5 now says a BurlerReview row naming a fan setting records its round's fans whatever the value, that absent means only solo or single-fixer and no profile conflict, and that a BurlerFix row naming `fixer_fan_setting` requires that recorded round.
  The acceptance scenario now reads "runs solo and still records the round, so its segment's fix row runs the single fixer from the record".
- **F2 (LOW)** — fixed as proposed.
  D8 names each source of the Darn-Review entry: budget, hard cap and models from `darn.yaml`, empty model keys falling back to `loom.yaml`'s shared lists handed over directly, and Bouncer models from the webster judge pair.
- **F3 (LOW)** — fixed as proposed, with a self-check addition.
  D8 requires the status path's budgets to come from the same perch table and per-perch resolution as `Segments`, and `Routing`/`DarnRouting` refuse a segment missing from their budget map.
  A new acceptance scenario requires status routing and the built shed to agree on MaxBounces.
  The self-check added this refusal's way forward and a scenario that reaches it (see Self-check).
- **F4 (LOW)** — fixed as proposed.
  D6's shape refusal now exempts null fans.
  D5 says the null rule applies to what `Setting` returns, and that on a hub run the module loaders refuse a null or empty-list fan first.
- **F5 (LOW)** — fixed as proposed.
  The Notes now require `envkit.FullEnv` to fill `Segments` for every embedded review segment, and `Setting` with a lookup that answers any registered module's template key.
- **F6 (LOW)** — fixed as proposed.
  The PATTERN changes entry now reads "a resolved one of a review segment's row is a `SegmentSettings` field".
  The record holds no separate declared PATTERN text, so there was no second copy to change.
- **F7 (NIT)** — fixed as proposed.
  D4's Rationale now says the snapshot is a later read than the wire-time loader read, and that burler refuses a fan that no longer resolves when the round starts.
- **F8 (NIT)** — fixed differently.
  I did not make the proposed change, naming both the perch key and `review_max_bounces` as the keys to check.
  The self-check showed it names the same key twice for Darn-Review and points a loom per-perch refusal at a run-wide key that was never the source.
  `reviewbudget.Resolve` already records the source key in `Caps.SoftKey`/`HardKey` (`internal/reviewbudget/reviewbudget.go:55-65`).
  So D1's `SegmentSettings` now carries the config key and file each of the budget and hard cap resolved from.
  D8's refusal names the segment and that key and file, and the matching acceptance scenario was updated to match.
- **F9 (NIT)** — fixed as proposed.
  The restating Constraints bullet is deleted.

## Self-check

- fix-claims — no findings.
- fix-consistency — no findings.
- fix-way-forward, MEDIUM: the new D8 missing-segment refusal had no way forward and no scenario reaching it.
  Fixed: the refusal names the way forward (add the segment's row to the wiring's segment-to-perch table), an acceptance scenario reaches it, and the `PATTERN-refusal-way-forward` constraint now covers D8.
- fix-way-forward, LOW: the F8 refusal named keys without their files, so it misdirected a Darn-Review refusal to `loom.yaml`.
  Fixed, merged with the fix-proposal finding below.
- fix-proposal, LOW: F8's proposal names both candidate keys even though the resolution records the exact one.
  Fixed with my own change, as described under F8.

## Disputed

None.

## Deferred

None.
