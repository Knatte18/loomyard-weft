# Cut the existing redundant tests of every package, found by the coverage maps

`redundancy-sweep-existing` · Note · enhancement, testkit, gate

## Seen (orch, 2026-11-09)

The coverage maps (`internal/testcover`, built by `lym gate testmap`) record, per package, the blocks each tier 0 and tier 1 test covers.
The gate report (the advisory redundancy step of `lym gate test`, landed as `7f21142e`) judges only the new tests of a diff against the map; nothing yet uses the maps to find redundant tests that already exist.
The operator wants the existing ones cut, across the packages, not only kept from growing.

## Idea

1. A candidate list per package: every test whose covered blocks the package's other tests already cover, with its run time, using the same judgment as the gate report.
   Tests marked `//testtiming:keep` are left out.
2. Cut tasks on that list, slowest packages first, each run delegating the per-test reading to subagents:
   a candidate is cut or folded into another test only when its assertions add nothing the covering tests check, since equal coverage does not mean equal checks.
3. `internal/fabricengine` was swept by hand on 2026-11-08 and is being reworked by its serial-phase tasks, so it comes last or not at all.

## Changed by the Sketch `testmap-upkeep-design` (board tidy, 2026-11-11)

Its approved decision record (D13) asks two changes of this note:

- Before judging a package, the sweep runs a hand call of `lym gate testmap` naming it (D9), which rebuilds that package's map whole.
- Spawn-marked tests (D4) are neither candidates nor covers.

## Uses today's maps, now (operator, 2026-11-15)

The cut does not wait for the maps' check-in (`testmap-maps-in-weft`) or for the payoff measurement (`testmap-payoff-measure`).
It uses the maps that exist today, stored beside the repository under `<bolt>/.lym/testcover/`:
a full set of 211 maps, built on 2026-11-12 by the upkeep pass in 44 minutes.

What it builds on:

- The candidate list per package: `lym gate redundant <package>` (commit `79f0eb4a`, task `test-redundancy-candidates`) prints a package's candidates from its map as JSON, with tier, seconds, covering tests and whether each is in the removable set. It omits `//testtiming:keep` tests and spawn-marked tests.
  It runs no test and takes no gate slot.
- The pilot report `docs/research/speed/coverage-map-pilot-2026-11-12/report.md` ran it on `internal/adoptcli` only:
  of 30 judged tests, 2 were candidates, and both assert something their covering tests do not, so the map is a reading filter, not a verdict.

File it as tasks once the operator has read the candidate lists.

## Built since (tidy, 2026-11-15)

Task `testmap-on-command` (`72b26d1a`) retired the upkeep pass.
No dobby or landing keeps the maps fresh now, so the 211 maps above go stale as landings change their packages.
A sweep of a package first runs `lym gate testmap` on it by hand, as D9 says; that call now waits for the package's build lock and has no default package cap.
