# Refuse a card id ending in the fabric suffix at the plan gate

Fixes #874.

A card slug ending in `-weft`, such as `6-clone-records-marker-on-prime-weft`, made every command naming the card a fabric reference to the implementer audit.
`ReferenceRule`'s bare-word pattern reads any unquoted word ending in the suffix as a weft sibling worktree, so a fork's own `lyx gate test --card <card>` failed the batch with a fabric-reference finding, and `recover-batch` then refused with `needs_fresh`.

## What changes

- `planparser.CheckCardFabricReference` now runs the matcher's path rule over each card id (`N-<slug>`).
  A hit is a `card-fabric-reference` finding naming the card and the matched id, with the way forward: rename the card in the Card Index and its file name so its slug no longer ends in the fabric suffix.
- `contracts/specs/loom-plan-spec.md`'s `card-fabric-reference` entry and the `internal/planparser` package doc describe the new hit.
- `TestCheckCardFabricReference` gains two rows: a slug ending in the suffix fails naming the card and the rename, and a slug holding the suffix mid-name passes.

## Design decisions

- **Refuse at the plan gate, leave the audit alone.**
  A card id appears in many commands a fork runs besides `--card`: paths under webster's state, commit messages, grep patterns.
  Refusing the name once covers every spelling, and the audit keeps every guard it has, without learning one CLI's flag grammar.
- **Same check ID.**
  The hit uses the same matcher and the same fix site as the existing `card-fabric-reference` scan, so it adds no row to the spec's ordered check list.
- **No fabric naming in planparser.**
  The check reaches the suffix only through the `FabricReferenceMatcher` it already receives.
  The test derives the suffix from `fabricengine.RecordsWorktree` rather than importing `weftname`, which the fabric-vocabulary scan forbids outside its owner set.

## Risk

`CheckCardFabricReference` takes no done set, so a rework of a plan approved before this change whose done card carries a `-weft` slug is refused too.
Such a plan predates the check and is expected to be rare.
