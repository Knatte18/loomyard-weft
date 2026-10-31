```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the change set still agrees with the sibling cards, the overview and the decision record.

**1. LOW — card 2, ImpactSummary is stale.**
- **Changed text:** Card 2 now has `armFromSeed` add a re-seed way forward to every failed recipe lookup.
- **Problem:** The ImpactSummary still reads "One decode branch and its constant go; only a seed recording `beam` changes reading, and no live seed records it." It doesn't mention that every `lym shed` unknown-recipe refusal now carries new way-forward text.
- **Evidence:** In card 2's Intent, `armFromSeed` appends `; way forward: remove the run's seed.json and seed the run again with "lym shed seed <run-id> --recipe <recipe>"` whenever a lookup fails, not only for `beam`. The test rows it pins seed `bogus-recipe`.

**2. LOW — card 2 calls its route the same as card 19's, but the wording differs.**
- **Changed text:** "the re-seed route card 19 gives `lym loom`" in card 2's Intent, and "with the same re-seed way forward" in the card 19 cross-reference.
- **Problem:** The two way-forward strings differ.
- **Evidence:**
  - Card 19's text, `19-loom-verbs-loom-only.md:30`: `remove the run's seed.json and seed the run again with "lym loom start" or "lym shed seed <run-id> --recipe <recipe>"`.
  - Card 2's text names `lym shed seed` alone.
- **Suggested fix:** Either word card 2's claim as the same re-seed step, or carry over card 19's exact text.

**3. LOW — card 6, ImpactSummary has not caught up with the fix.**
- **Changed text:** Card 6 now changes `reviewbudget.Values` and `CheckResolved` (new `RunWideSpec`), changes the signature of `twillLenses`, and adds several test and spec targets.
- **Problem:** The ImpactSummary still reads "loom's loader fills them from the new module so its readers keep their signatures, and recipe setting references, segment settings, reconcile moves and run-settings translation follow." It says nothing of the `reviewbudget` change or the `twillLenses` signature change.
- **Evidence:** Card 6's Edit list now holds `internal/reviewbudget#Values`, `internal/reviewbudget#CheckResolved` and the arrow target `internal/loomengine#twillLenses -> func twillLenses(dirs configengine.Dirs, fan, module, key string) ...`.

Checks with no finding:
- No other card or the overview names the removed `TestRecipe_DriftVerifyPrecedesCodeReview`, or the old "non-loom recipe" wording for a `beam` seed.
- Card 18 line 38 now agrees with the `TestWire_BeamRecipe` subtest.
- Card 6's `TestPerchHardCap` rows agree with the new routing of `seedLoomConfigWithKeys` by template.
- The decision record's D11 body (line 192) and its acceptance criterion (line 303) still say `lym loom` refuses `beam` as a non-loom recipe. The Added after Discussion entry supersedes that text, the fix doesn't touch it, and it is out of scope.

Tool calls:
1. Grepped the other plan cards and the overview for references to the changed targets and phrases. My context did not hold cards 1, 3–4, 7–9, 11–15, 19–20 or the overview.
2. Grepped the overview and the decision record for `beam`, Drift-Verify and cap mentions. This was to check card 2's agreement with D11 and its Added entry, whose full text my context held only in part.
