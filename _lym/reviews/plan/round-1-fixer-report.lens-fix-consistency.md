```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency, checking that the fixer's change set agrees with the cards, decisions and cross-references around it.

**LOW: card 6's arras row leaves the test's own doc and error text naming only three recipes**
- Changed text: `06-loom-recipes-motif-arras.md`, Tests bullet "`TestInterruptPolicies_MatchAssembledRows` adds `shedrun.RecipeArras` to the recipes it builds."
- Summary: the card adds arras to the loop, but the test's doc comment and its reverse-direction error message stay as they are, and both still list the recipes by name.
- Evidence: `internal/loomrecipe/interruptpolicy_meta_test.go:18` says "covers exactly the rows New assembles for the tapestry, darn and sampler recipes". `:40` says "which none of tapestry, darn and sampler has". Neither leaves the card's Intent.

**LOW: card 6's ImpactSummary leaves out the new loomshed edit**
- Changed text: `06-loom-recipes-motif-arras.md`, the Edit entries `internal/loomshed#InterruptPolicies` and `internal/loomrecipe#TestInterruptPolicies_MatchAssembledRows`, and the Intent paragraph that maps `NameMotif` to `shedverbs.InterruptPolicyReinvoke`.
- Summary: the ImpactSummary still describes only the recipes and loomrecipe, though the card now edits a loomshed table.
- Evidence: the ImpactSummary reads "Two new embedded recipes and one name table in loomrecipe; every accessor now expands…". It says nothing about the `loomshed.InterruptPolicies` entry, which `shedverbs`' envelopes read.

**LOW: card 4's spec edit adds `Expand` but leaves nearby spec text unchanged**
- Changed text: `04-shedbuild-expand.md`, `contracts/specs/shed-recipe-spec.md` under Extends, and the Intent line "`shed-recipe-spec.md` gains an expansion paragraph naming `Expand`, the told resolver, the block exit and the block path."
- Summary: the expansion paragraph lands in a section whose sentences go stale once it is there:
  - **Four exported functions** still lists `Parse`, `Load`, `Build` and `Check`.
  - **Validation split** still says the package "runs no reachability, cycle, … analysis of its own". `Expand` refuses a self-including block chain, a cycle.
  - Card 5's `RoutingFrom` and its rule that `Build` takes the expanded recipe never reach the spec, and card 5 does not list the spec.
- Evidence: `contracts/specs/shed-recipe-spec.md:93` reads "**Four exported functions.**" and `:112-113` holds the Validation split paragraph. Card 4 Intent lists the refusal "recipe %q includes itself through &lt;chain…&gt;".

Card 3's spec line agrees with `checkRowShape`'s block-row key set. Card 7's Edit/Uses lists, Intent and Tests agree after `TestWire_DefaultConfig` and `loomSegmentPerches` were removed. Card 5's new test entry agrees with card 2's `BlockGateAlways`. Card 6's trimmed TestBlockRecipes bullet still agrees with card 5's `TestRoutingOf_MatchesNewShedRowForRow`. Card 8 was not changed.

Tool calls:
1. Grepped `00-overview.md` and the other cards for references to the changed items. My context did not hold the overview card or the cards' spec and doc entries.
2. Grepped `00-overview.md`'s decisions for D8 and D10 and listed `_lym`. I needed the decision text the changes depend on, which my context lacked.
