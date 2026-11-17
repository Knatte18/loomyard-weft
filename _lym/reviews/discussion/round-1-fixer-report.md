# Fixer report — discussion round 1

Every finding's premise held against the code; all eleven are fixed as proposed in `_lym/discussion/decision-record.md`.

## Changes

- F1: D4 refuses a hub whose tier 0 or tier 1 sets `memory_mb` below `redundancy_test_memory_mb`, naming the tier, both values and `lym config gate` as the way forward.
  D1 lists the refusal, the `PATTERN-refusal-way-forward` constraint names D4, and a D4 criterion covers it.
- F2: D3 states map mode builds no `lym`, citing `lymbin` as the one reader of `LYM_PREBUILT_LYM` and the spawn scan; the line-179 criterion is deleted.
- F3: D4 runs a package's tier 0 tests before its tier 1 tests, each up to its own clamped count.
  The Bound names tier 0's `parallel` as the builds' `-p`, and the criterion names the running tier.
- F4: D6's first check runs once the build has listed the linked packages and compiled; a Bound states the unseen-edit window, and the Rationale no longer claims a mid-run edit cannot reach a map.
- F5: D1 refuses `--stop-on-fail` and `--dir`/`-C`, applies `--main-packages` with `--card`, and its Bound names what map mode skips and what bounds that; the criterion lists both flags.
- F6: D1's package set takes in every package of the scope that has a map; a package with no mapped test gets a hash-only map; a D2 criterion covers a deleted last test.
- F7: D9 retargets every rebuild-naming refusal of `lym gate redundant`; Scope and the D9 criterion follow.
- F8: D6 and its criterion use package ownership down to the next package directory or nested module.
- F9: D7 writes the map of every package D6 lets through.
- F10: the `PATTERN-gate-slot` constraint names the map run in the background's slot rule; Scope and the Review criterion name `pattern/PATTERN-gate-slot.md`.
- F11: D5 gains a Bound that a test passing only alone stands as a plain pass, traced only in the call's output and log.

## Self-check

- fix-consistency, LOW, support log still says D3 builds `lym`: fixed as proposed, the support-log entry records the round-1 reversal; Q6 in the ledger also updated to D6's new check order.
- fix-consistency, LOW, no removal list names the measure loop's `lym` build: fixed as proposed, D9 lists it among what goes.
- fix-claims, LOW, D3's spawn-scan claim misses tier0 files: fixed as proposed, D3 says a tier0 test spawns nothing and the scan marks every other test.
- fix-way-forward, MEDIUM, the stale-files refusal's way forward meets D6's dirty-package skip: fixed as proposed, D9 has that refusal say to commit or revert the edits first.
- fix-proposal, MEDIUM, D6's Bound misses edits before the first check: fixed as proposed, the Bound names both windows.
- fix-proposal, MEDIUM, D1's Bound says the run writes only maps while D7 writes a log: fixed as proposed.
- fix-proposal, LOW, D6 has no check point for a package with no build: fixed as proposed, its one check runs before its map write.
- fix-proposal, LOW, a build whose listing fails never reaches D6's check: fixed as proposed, D6 says it records its failure with no check.

## Disputed

None.

## Deferred

None.

## Rulings

None.
