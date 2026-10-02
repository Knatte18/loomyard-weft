# Agent naming run stopped after batch 01

Batch 01 (`agentname-package`) finished as `done` at `efa6ec32`.
It added `internal/agentname` (grammar, tests, seam-enforcement test) and the Agent Name Invariant in `CONSTRAINTS.md`.
The digest reported no deviations, only informational scope-outside-plan warnings for the new package's own symbols.

Batch 02 (`fabric-code-record`) never started.
After batch 01 was recorded, `begin-batch 02` first returned an envelope with advisories.
Later calls to it refused with `plan_drifted`.
The refusal named two blocking findings:

- `glyph-not-found/3-fabric-code-verb`, which targets `internal/fabricengine/codebinding.go#`.
- `glyph-not-found/6-geometry-naming`, which targets the same file and unit.

Card 02 creates that file, so cards 03 and 06 reference a unit that does not yet exist in the tree.
An operator should edit the plan so these cards match the tree.
They then run `lyx webster rebaseline --card NN` for each edited card and resume with `lyx webster run`.

I made one extra `begin-batch 2` call after the first refusal, to read the message, before stopping.
