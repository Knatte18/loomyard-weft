# Fixer report — discussion, round 2

## Changes

- F1 (BLOCKING), fixed as proposed, with the directory rule.
  Premise confirmed: `Glyph.UnitPath` in quarry v0.2.1 returns `g.Unit` unchanged for Go, the package directory for a package unit.
  D4 now states that a file-unit target covers its one file by equality and a package-unit target, every Go symbol target among them, covers every file directly in its package directory, never a subdirectory.
  The Bound names what that lets through, non-Go files in the directory included, and both acceptance criteria on the exemption are reworded, the import-only one with a subdirectory negative case.
  The plan-index alternative is recorded under the support log's rejected alternatives.
- F2 (BLOCKING), fixed as proposed.
  Premise confirmed: `contracts/recipes/recipes.go` embeds `loom-recipe.yaml` as `LoomRecipe`, and `peerLine` in `internal/loomcli/peers.go` carries no row order.
  D5 now reads the order from `lyx loom peers`, which gains `row` per peer line and `own_row` from a task pair, both from the embedded recipe, 0 for a producer that is not a loom row.
  The Bound states the fields are read-only and reveal only recipe positions; Constraints gains the CLI output change with its doc in `internal/loomcli/doc.go` (no verb, so `docs/overview.md` is unchanged); a criterion pins the source in a hub with no loomyard checkout.
  A plan-writer note names `peerLine` and `recipes.LoomRecipe`.
- F3 (MEDIUM), fixed as proposed.
  D2's Bound restricts "keeps its eligibility" to Merge-In and the other rows and names the Publish narrowing; a criterion pins a blocked Publish stop of another cause with a stale record as eligible.
- F4 (MEDIUM), fixed as proposed.
  D5 states that a design conflict with a landed parent commit escalates, the agent comparing the commit's subject and diff against this run's decision record and diff; a criterion pins it.
- F5 (LOW), fixed as proposed, by the Bound option.
  D4 states the symbol names leave every reader surface, and its Bound states the file granularity; the digest's `deviations` shape stays `[]string`.
- Support log: round 2 entry, Q10 revised, two rejected alternatives added.

## Disputed

None.

## Deferred

None.
