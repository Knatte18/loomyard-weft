# Round 2 fixer report

## Changed

All edits are in `_lyx/discussion/decision-record.md`.

- F1: D10's Bound now names `LoadOrTemplate` as the kept loader, states the absent-file and present-invalid-file behavior, and places both value checks in `orchengine.LoadConfig`; the PATTERN-config-strictness Constraints line matches.
- F2: D7 gained a `Bound:` line naming the lost-completion edge, `cycle_mode: clear` as the way back, and the smoke assertion.
- F3: D13 states one file per notice written atomically, the appender-checked cap with its transient overshoot, and at-least-once delivery; the Bound line's wording follows.

## Deferred

None.
