# Card gate (--tier 0) does not vet a card's packages under the other tier tag sets, so helper dependencies across tagged files fail only in the batch gate

`card-gate-vets-all-tag-sets` · Task · bug, gate, webster

Depends on: [`gate-runs-past-failing-tier`](design-gate-runs-past-failing-tier.md)

Imported from [issue #865](https://github.com/Knatte18/loomyard/issues/865).

What happened: while retagging test files across tiers, the card gate (`--tier 0`) did not compile the card's packages under the tier1, tier2 and tier3 tag sets.
A helper defined in a retagged tier0 file and used by a tier1 or tier2 file failed only in the batch gate's vet steps.
Each card gate passed while the batch gate found about twenty such helpers across the cards, and fixing them cost a separate pass.

Why abnormal: the card gate is meant to catch a card's own breakage, but a tag-set compile break is invisible until the later batch gate, far from the card that caused it.

What to do: make a card's gate step also vet the card's packages under the other tag sets (at least when the card touches build tags), so such failures surface at the card.

Provenance: task gate-runs-once; notes 24-retag-fabricengine.md