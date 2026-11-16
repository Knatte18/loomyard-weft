# Fixer-report, code review round 1

## Changes

- F1 (MEDIUM), fixed as proposed: `pattern/PATTERN-test-speed.md` now says `lym gate testmap` rebuilds at the HEAD of the worktree it runs in, and plans every map at that HEAD.
  Commits `ba2a118c` and a follow-up that names the anchor's `.lym` (see Self-check).
- F2 (LOW), fixed as proposed: `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps` builds the hub with `Anchor: "backend"`, seeds the module under it and asserts the map lies under `testcover.Dir(anchor)` at the worktree's HEAD. Commit `a040f962`.
- F3 (LOW), fixed as proposed: the `testmap` Long restores the audit-round sentence and folds "It runs in any worktree of the hub." into the next sentence, with a few trims so it stays under `maxLongBytes`. Commit `8e4f4270`.

Gate: `lym gate test ./cmd/lym ./internal/gatecli` passes at tier0, and `--tier 1 ./internal/gatecli` passed, including the changed test.

## Self-check

All four forks returned a report; their files sit beside this report as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW, PATTERN bullet said "under its `.lym`" and read as three map locations: fixed with own change, reworded to "into the hub's shared maps from the prime and into the worktree's anchor `.lym` elsewhere".
- fix-proposal, LOW, same loose "under .lym" in the `testmap` Long: not changed, the Long sits within about 30 bytes of the 2000-byte cap and the clause "its own under .lym" stays true; the gatecli package doc carries the anchor detail.
- fix-proposal, LOW, the "over the whole module before a sweep" advice was cut from the Long: tried restoring it, the Long became 2027 bytes against the 2000 cap (`cmd/lym/clitree_test.go:413`), so reverted. The advice lives in the package doc, which the cap's own message names as the place for it.
- fix-proposal, F2 does not assert the worktree root holds no map: not a finding, review asked only for the anchor pin.

## Disputed

None.

## Deferred

None.

## Rulings

None.
