# Fixer report — discussion round 1

Both findings' premises hold against the code: `internal/parentreview/gate.go` passes the discussion with a Warn once `WaitBound` elapses, `burlerengine.Config` holds only `Lenses` and `Fans`, and `batcher` and `shuttleengine` register open maps through `ConfigOpenMaps`.

## Changes

- F1 (BLOCKING), fixed as proposed, in `_lyx/discussion/decision-record.md`:
  - D4's guard category reads "switches off or hollows a guard, including a review or an approval gate, a timeout whose expiry lets that gate pass, and the prose or lens set a reviewer is given", and a new sentence says the test asks what a key's value can do to a guard, not what kind of key it is.
  - D4 adds loom's `LocalDenied: parent_review_wait_min` and burler's `LocalDenied: lenses, fans`, each with its reason, notes that burler's local layer therefore holds no key, and says "the remaining per-worktree modules deny none".
  - D4's Bound says "switches off or hollows" an agent guard, a review or an approval gate.
  - D2 says only `models` takes template-lacking keys in the local layer; D5's exemption and the acceptance row narrow to `models` aliases.
  - D8's validation and the `lyx board upsert` acceptance row replace "seed-only module" with "`models` alias", so the board write agrees with D5 and D4.
  - The refused-local-keys acceptance row adds `loom.parent_review_wait_min` and `burler.lenses`.
  - The `PATTERN-config-layers` constraint carries the extended category wording.
  - The auto-mode assumption on D4's narrowing names loom's and burler's denied keys.
- F2 (MEDIUM), fixed as proposed: D5 exempts a key path under one of the module's open maps (`configreg.Module.OpenMaps`) from the template-lacks refusal, matching D8, and a new acceptance row says a local file and `run_settings` accept an open-map entry the template lacks and the module loads.
- `_lyx/discussion/support-log.md`: a Round 4 entry records both findings, and Q5 names the extended guard category.

## Disputed

None.

## Deferred

None.
