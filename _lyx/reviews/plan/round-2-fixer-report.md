# Plan round 2 — fixer report

## Changes

- F1 (BLOCKING), fixed as proposed: card 1 lists `internal/burlerengine#TestConfigTemplate_DecodesAndIsSelfConsistent` under Edit.
  Its "needs no change" sentence is replaced: the test raises `wantLenses` and `wantFans` by the card's lenses and fans, adds the four new fans to its per-fan entry-count map, and keeps the defined-lens check.
  The ImpactSummary names the pin change.
- F2 (MEDIUM), fixed as recommended: card 1's Intent drops "written as emphasis, never exclusion".
  The new lenses keep the bench text ending in its `Your list:` sentence, per D4's "a fork reports only what its lens's procedure produces".
  The test's emphasis-suffix assertion skips the lenses the four new fans name, and its `ignore ` check still covers every lens; the bench lens texts hold no `ignore `.
- F3 (BLOCKING), fixed as proposed: card 8 lists `internal/configreg#TestModules_EveryTemplateKeyClassified` under Edit.
  Its Tests paragraph says loom's `wantLocalDenied` entry gains `discussion_fix_fan` and `plan_fix_fan` in sorted order, each right after `discussion_fan` and `plan_fan`.
- F4 (LOW), fixed as proposed: card 8's `TestLoadConfig_Refuses` sentence adds a row whose `plan_fan` list holds a mapping entry, reaching `fanShapeError` through `FanList.UnmarshalYAML`'s list branch.

`lyx loom validate-plan` passes with informational findings only.

## Disputed

None.

## Deferred

None.
