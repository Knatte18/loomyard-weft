# Fixer report — plan round 3

## Changes

- **F1** (MEDIUM, fixed as proposed): `_lyx/plan/01-production-fans.md` now gives the list-sentence ruling to the seven reviewer lenses by name.
  A new sentence says `fix-consistency`, `fix-claims` and `fix-way-forward` keep the twill-fixer bench text with no list sentence, `fix-affected-set` ends by listing each finding it left unchecked (D8), and `fix-proposal` is a procedure with no list sentence.
  The no-emphasis-line sentence stays, with D4 as the reason.
- **F2** (LOW, fixed as proposed): card 1's Intent now says the header comment of `internal/burlerengine/template.yaml`, from "Lens prose is always phrased as EMPHASIS" onward, is edited: the dormant lenses are emphasis prose, the production fans' procedure lenses end in their own list instead, and the no-"ignore" rule holds for every lens.
  The ImpactSummary names the header comment.
- **F3** (LOW, fixed as proposed): card 1 drops the `internal/burlerengine#TestResolveFan` Edit target and its row sentence.
  `TestConfigTemplate_DecodesAndIsSelfConsistent` now also adds the four new fans to its `wantOrders` map, which resolves each through `ResolveFan` against a zero `Config`.
  The ImpactSummary no longer mentions a new test row.
  No other card references `TestResolveFan`.
- **F4** (LOW, fixed as proposed): card 8's `TestLoadConfig_Refuses` sentence now says the existing "unknown discussion_fan" and "unknown plan_fan" rows keep their key, fan name and `known fans: ` assertions but replace `to empty to run the segment solo` with `entry 1` and `set the entry to ""`.
  The malformed-burler.yaml row keeps its assertions.

## Disputed

None.

## Deferred

None.
