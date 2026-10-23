# Fixer report — plan, round 2

## Changes

- **F1 (BLOCKING), fixed as proposed**, `_lyx/plan/19-arbiter-amendment-target.md`.
  Premise confirmed: `goDriverDepsFor` calls `c.arbiterDepsFor()` once per drive (`internal/loomcli/drivewiring.go:129`), and `Judge` reuses those deps for every stop.
  `arbiterDeps` literals exist only in `arbiterDepsFor` and `arbiterFixture.deps`, and the fix covers both.
  The Edit list gains `arbiterDeps`, `arbiterCall.settleFresh`, `TestArbiterVerb_SpawnPath`, `arbiterFixture` and `arbiterFixture.deps`.
  In the Intent, the `arbiterDepsFor` sentence is replaced by the three proposed sentences: an `AmendmentTarget func() string` seam read at each spawn, `settleFresh` filling `promptInput.AmendmentTarget`, and `arbiterDepsFor` reading the recipe target into a local (which fixes the two-value call) and setting the closure.
  The Tests paragraph gains the fixture field and the `TestArbiterVerb_SpawnPath` row, and the ImpactSummary has the proposed wording.
  The self-check changed how the row is worded; see below.
- **F2 (LOW), fixed differently**, `_lyx/plan/24-planparser-target-holds.md`.
  The proposed clause said the containment is decided "as `columnsOverlap` decides it".
  I did not make that change as worded: `columnsOverlap` swaps the two chains so the shorter comes first (`tools/plandag/graph.go:131-134`), which makes it symmetric, so following it literally would let a method target `pkg#T.M` hold the member `pkg#T`.
  The Intent now reuses `columnsOverlap`'s parsing and chain comparison (`ParseRef`, `ParseMintedCSharpID`, no string prefix, PATTERN-glyph-conversion-chokepoint) and says the check runs one way: same unit, with the target's owner chain plus name a prefix of `member`'s and never the reverse.
  The Uses list gains `internal/planparser#ParseRef` and `internal/planparser#ParseMintedCSharpID`.
  `TestTargetHolds` gains a false row: a member target against its owning type as `member`.

`lyx loom validate-plan` passes with only card 25's informational create-new-unit findings.

## Self-check

- fix-affected-set: no findings.
- fix-consistency, LOW: card 24's Uses list was missing the two parsers the Intent now names. Fixed as proposed.
- fix-claims, MEDIUM, and fix-proposal, MEDIUM, merged: `columnsOverlap` is symmetric, while `TargetHolds` runs one way. Fixed as proposed: the one-way rule and a reverse-direction false row are now in card 24.
- fix-claims, LOW, and fix-proposal, LOW, merged: card 19's new test row could not tell "when the call is built" from "when it spawns", since only `settleFresh` reads the seam (`internal/loomcli/arbiter.go:211-246`). Fixed with fix-proposal's suggested wording: the fixture answers `Plan-Burler-Review` only once the attempt record exists under `f.attemptDir(1)`. The row now also says that it never reaches `arbiterDepsFor`.

## Disputed

None.

## Deferred

None.
