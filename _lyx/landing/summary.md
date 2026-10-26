# Refuse a card slug ending in the weft suffix at the plan gate

A card whose slug ends in the fabric suffix, such as `6-clone-records-marker-on-prime-weft`, made every command naming the card a fabric reference to the implementer audit.
The fork's own `lyx gate test --card <card>` run was enough to mark its batch failed, although it touched no fabric repo.

`CheckCardFabricReference` in `internal/planparser` now also runs the matcher's path rule over each card id (`N-<slug>`).
A match is a `card-fabric-reference` finding naming the card and the matched text.
Its way forward is to rename the card in the Card Index and its file name so the slug no longer ends in the suffix.
A slug that only contains `weft` without ending in the suffix, such as `weft-prime-marker`, passes.
`contracts/specs/loom-plan-spec.md` and the `internal/planparser` package doc describe the new hit.

Closes #874

## Design decisions

- The name is refused once at the plan gate, not handled in the audit.
  A card id appears in many commands besides `--card` (state paths, commit messages, grep patterns), and the audit reads every one, so refusing the name covers every spelling.
  `internal/fabricengine/refscanner.go` is unchanged and keeps every guard it has.
- The hit reuses the `card-fabric-reference` check ID, since it is the same rule with the same matcher and the fix lives in the same place; no row is added to the spec's ordered check list.
- `internal/planparser` reaches the suffix only through the `FabricReferenceMatcher` it already receives, so it still imports no weft naming.

## To know

- The check takes no done set, so a rework of an already approved plan whose done card has a `-weft` slug is now refused.
  The operator renames the card or accepts the batch audit by hand.

## Tests

`TestCheckCardFabricReference` gains two tier0 rows: a slug ending in the suffix fails naming the card and the rename, and one holding the suffix mid-name passes.
The test was not run by hand outside the gate.
