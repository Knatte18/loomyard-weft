# Round 1 fixer-report — plan

## Changes

- **F1** (LOW, consistency), fixed as proposed.
  `_lyx/plan/16-single-seed-parser.md:23`: the bound now names `ReadSeed` as what already refuses the shut-out seeds, and says no lyx writer produces one: `WriteSeed` refuses an unknown driver and `lyx shed seed` refuses an empty `--param` key.
  The premise checks out against the code: `WriteSeed` (`internal/shedrun/seed.go:174`) validates run id, recipe and driver but not params keys, and `parseSeedParams` (`internal/shedcli/seed.go`) refuses an empty key.
  The other `WriteSeed` callers (`loomSeedFor`, battencli's `arm` seed and `childSeedFor`) write fixed params keys, so the claim that no lyx writer produces such a seed holds.
  The sentence is split at its semicolon, one clause per line, following the semantic-line-break rule.

## Disputed

None.

## Deferred

None.
