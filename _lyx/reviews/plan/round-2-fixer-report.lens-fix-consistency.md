```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency: I checked whether the change set agrees with its sibling cards, the cross-references and the Card Index.

no findings

What I checked:
- **Renumbered references.** After the insert, every `card N` reference in the cards points at the right card:
  - card 2 → 12;
  - card 9 → 12;
  - card 11 → 10, 7 and 10;
  - card 12 → 9, 10, 10 and 9;
  - the new card 6 → 11 for the inert check.
- **No leftover twin.** The `impactset` Create, Extends, Intent and ImpactSummary text left card 7 completely. The new card 6 holds it. Card 7's Uses names both symbols. The Card Index lines for 6 and 7 agree.
- **Card 11 Uses `plan:internal/impactset#ChangedPaths`.** That symbol is now created in card 6, which comes earlier, so the order still holds.
- **Card 9's seed tree.** "Name `Circle`" fits card 5's rule, because naming works through each file's `Identifiers` (card 4). "`OtherTests` names neither `Circle` nor `Unused`" keeps both the unrelated-class row and card 12's `Unused.cs` row valid. `[TestFixture]` and `[Test]` match card 5's `TestAttributed` and `HasTestMethod`.
- **Card 8.** The new `TestFilterArgument` sentence agrees with the refusal paragraph: it names the tier and the `lyx config gate` way forward. `renderTierCommand` passing `runtime.GOOS` matches how `quoteAll` works today. The `TestRenderTierCommand` rows and the ImpactSummary are untouched and still true, since `renderTierCommand`'s signature did not change.

Further tool calls:
1. Grepped the plan cards for `filterArgument`, `impactset`, `Circle`, `OtherTests` and `TestFixture`, and for card-number mentions in the overview and cards 8, 9, 10 and 13. My context held only the reference lines the insert reported and my own earlier grep over the old numbering.
2. Read card 5's selection rules and card 4's `Identifiers` model. My context lacked the test-class and naming rules that the new seed-tree sentence depends on.
