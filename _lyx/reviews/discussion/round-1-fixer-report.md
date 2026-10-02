# Fixer report: Discussion-Review round 1

All six findings fixed; `lyx loom validate-discussion` exits 0 afterwards.

## Changed

- **F1 (MEDIUM), fold reverts newer status.**
  Decision 9 now applies each legacy done mark once, tracked in a `legacy_done` slug list that migration seeds with every legacy slug already `done`;
  the rationale no longer claims the memoryless fold cannot revert a status.
  Decision 1 and Scope name `legacy_done` in the store shape, Decision 10 drops it on `retire-legacy`, and two fold acceptance criteria cover the revert and slug-reuse cases.
  The support log adds the memoryless fold to Rejected alternatives and notes the change on ledger item 4.
- **F2 (LOW), tier rule and done dependencies.**
  Decision 7 exempts a `done` dependency from the tier comparison; a validation criterion checks it; Rejected alternatives and ledger item 8 record it.
- **F3 (LOW), defaults without checks.**
  Acceptance criteria add a Defaults item (tier 3, type `feature`, through `upsert` and `upsert-batch`), `abandoned` surviving `prune`, and an `abandoned` entry in the README golden test.
- **F4 (LOW), alias behavior.**
  Decision 13 states that the aliases see the whole store and that `promote-note` on an entry already at tier 1 succeeds without a change;
  the aliases acceptance criterion and ledger item 15 follow.
- **F5 (LOW), support log against Decision 21.**
  Interview turn 7 and ledger item 14 are marked as replaced by the hub review with the current resolution;
  the per-line-paragraph rejected alternative no longer argues for hard breaks, and hard line breaks are added as a rejected alternative with the hub's reason.
- **F6 (NIT), naming rule.**
  The "never Bolt" rule moved from Goal to Constraints without its provenance, which the Hub review entry already records; Goal keeps a clause naming the board as the tracker behind `lyx board`.

## Deferred

None.
