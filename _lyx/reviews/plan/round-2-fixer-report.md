# Fixer report — plan, round 2

## Changes

- **F1** (LOW, consistency), fixed as proposed in `_lyx/plan/08-circling-continue-hard-cap.md`.
  Premise confirmed: card 6's `LoadConfig` hard-cap refusals end with `then run "lyx loom resume"`, which the old `circlingWayHardCap` text stacked with "re-run the verb".
  - `circlingWayHardCap`'s Create signature becomes a format const whose `%s` takes the verb's slug: `const circlingWayHardCap = "way forward: fix the file or key the message names, then re-run \"lyx loom circling continue %s\" in place of any \"lyx loom resume\" it names"`.
  - The `circlingVerb` bullet states the slug fill and the new text, and adds one sentence on why the circling verb replaces card 6's resume step, citing D5.
  - The refusal-spec "circling: hard cap unresolved" bullet's way forward names `lyx loom circling continue <slug>` in place of any `lyx loom resume`.
  - `TestCirclingVerb_Refusals`' "hard cap unresolved" row also wants `lyx loom circling continue task-a` and "in place of any".
- **Gate `handle-malformed/8-circling-continue-hard-cap`** (blocking), fixed in the same Create entry.
  The first F1 edit wrapped the declaration in double backticks around a raw-string literal, which `moveLineRe` (`^`([^`]+)` -> `([^`]+)`$`) cannot match.
  The declaration head is now a single-backtick span holding an interpreted Go string with escaped quotes; its text is unchanged.
- **Gate `create-new-unit/3-bouncer-hard-cap`** (informational, two entries): no change.
  They report that card 3 creates `bouncer-fragment-decide-open.md` and `bouncer-fragment-decide-capped.md` as new units, which is what card 3 intends; neither reports a defect.

## Disputed

None.

## Deferred

None.
