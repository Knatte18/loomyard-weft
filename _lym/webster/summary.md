# Raddle recipe: usage records, cost report and the raddle run

All eight batches reached `done`. The digests listed no `departures` and no `renames` for any batch, and no digest carried warnings.

## Batches

- **01 shuttle-usage-split and 05 usage-wiring:** done before this session resumed.
- **07 report-config:** its first attempt had failed.
  `recover-batch` ran twice and ended `done` at `c0b09801`.
  The scope guard listed deviations in the raddle parser, the discussion splice, the loom raddle paths and the report cost files.
- **13 board-promote-raddle (cards 13–18):** done at `bff86847`.
  The raddle store, slug rule, replay, filing writes, `lym board raddle` and the Record-Write stored seed landed.
  Deviations covered `boardengine`, the `RecordWrite.ReadStored` seam fills in test wiring and `envkit`.
- **19 raddle-checks-and-state (cards 19–22):** done at `6a0eb4ef`.
  The raddle state, checks and gate, the `raddle-propose` and `raddle-approve` verbs, the Raddle-Write producer with its stencil, and the Raddle-File producer landed.
  Deviations covered `boardengine/raddlestore.go`, `loomcli` and `loomshed`.
- **23 raddle-recipe:** done at `35985ae7`.
  The recipe, its registry rows, the gate names and the `loomcli` wiring landed.
  Deviations covered `loomcli`, `loomrecipe`, `shedrecipe` conformance and fixtures, and `envkit`.
- **24 batten-raddle-start-guard (cards 24–25):** done at `56c68ed6`.
  Batten's start guard and Beam-Up's hand-off of the Sketch and proposal landed.
  Deviations: `battencli/startguard.go`, the two seam tests, `pawlengine/beamup.go` and `pawlengine/world_test.go`.
- **26 report-raddle (cards 26–27):** done at `7e402e0a`.
  `lym report raddle` and the glossary row landed.
  Deviations: `boardengine/raddlestore.go` and the `reportengine` raddle files.

## Scope guard

Every batch's deviations were informational, as the digests listed them; none made a batch stuck.

## Verify gate findings

The plan-level verify failed on attempt 1 with two failing tests:

- `internal/configsync.TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate`, the row where profiles under the recipe's former name are renamed in place and gain the template units they lack.
- `internal/boardcli.TestCLI/Profiles`.

No recorded card touched a failing package. One fixer fork was spawned for them.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `8889afb0db1f7176144e3362253156acbf02866a` fix: expect the raddle profile default in the profiles defaults assertions


## Plan rebaselined

webster: the plan changed since the run recorded it and was rebaselined on entry, keeping 3 begun batch(es) and accepting 07-report-config.md, 08-report-cost.md, 09-raddle-paths-and-keys.md, 10-raddle-proposal-spec.md, 11-discussion-part-splice.md, 12-raddleparser.md, 19-raddle-checks-and-state.md, 20-raddle-verbs.md, 22-raddle-file-producer.md, 23-raddle-recipe.md, 25-pawl-raddle-handoff.md, 26-report-raddle.md
