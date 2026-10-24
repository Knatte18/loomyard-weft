```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1, F2 and F3, which the fixer carried out as proposed. I looked for concrete cases where a proposal is wrong or breaks something else in the plan.

**F1 (card 8, `filterArgument(goos, ...)` and `TestFilterArgument`): no findings.**
- Only the tier's own `Filter` can hold a `%`. The terms are `FullyQualifiedName~` plus C# identifiers (card 5's `filterTerm`), so they never do. The refusal's way forward, "fix the tier's filter with lyx config gate", therefore always points at the right value.
- `renderTierCommand` passing `runtime.GOOS` matches how `quoteAll` quotes targets and args (`internal/gatecli/tiers.go:301`), so one command never mixes two quoting rules.
- `TestRenderTierCommand` rows assert POSIX single quotes on whatever OS hosts the test. That was true before this fix and the proposal does not change it.

**F2 (card 9 seed tree, `[TestFixture]` plus one `[Test]` method): no findings.**
- `[TestFixture]` sets `TestAttributed` and `[Test]` sets `HasTestMethod` (card 4, line 23), so each class is a test class under card 5, line 30.
- `CircleTests` naming `Circle` through a method body or field works: card 5's selection reads the file-level `Identifiers`.
- `OtherTests` naming neither `Circle` nor `Unused` keeps it unselected for both the `Circle.cs` row and the `Unused.cs` row in card 12.
- `CircleTests` is not stated never to name `Unused`. That is already covered by "`Unused` declaring a type `Unused` that no test class names", so nothing new breaks.

**F3 (new card 6 `impactset-diff-reads`): one LOW finding.**
- Changed text: `_lyx/plan/06-impactset-diff-reads.md`, which has no `Tests:` section.
- Summary: the card that now lands `ChangedPaths` and `AncestorBase` asserts nothing about them. Their behaviour is first exercised by card 7's `TestDeriveDiff` rows (an untracked `.cs` file counting as changed, a rename at both paths, a non-ancestor base), one card later.
- Evidence: before the split, those rows lived in the same card as the exports. After it, card 6's commit adds public surface used by other packages with no test, and its only check is that it compiles.
- The proposal itself said nothing about tests, so this comes from how the proposal splits the card. One way to fix it is a sentence in card 6 saying the two wrappers are covered by card 7's `TestDeriveDiff` and card 11's inert rows.
- Everything else in F3 holds. All the renumbered "card N" mentions, apart from the paragraph moved out of card 7, now point at the right cards, and `lyx loom validate-plan` reports ok true.

Tool calls I made:
- grep of cards 4 and 5 for the test-attribute and naming rules: my context held only card 8's text, not cards 4 and 5.
- grep of `PATTERN.md` and `pattern/` for an unused-export rule: my context had no PATTERN background files (found none relevant).
- `sed -n` of card 4, lines 24–26, and card 5, lines 22–34: needed the closure and test-class rules in full.
- grep of card 4 for the `TestAttributed` and `HasTestMethod` definitions: the earlier output cut off line 23.
- `ls` and grep of the burler stencils for a rule that every card has tests: my context had no plan review rubric (found none).
