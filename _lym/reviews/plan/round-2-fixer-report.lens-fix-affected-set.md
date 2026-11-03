```yaml
lens: fix-affected-set
outcome: reported
```

fix-affected-set: rerun F1's search for every place in the plan that names the wrong refusal for an unembedded recipe.

Unchecked findings: none. The review has only F1, and I reran its search.

**F1** (the set: every claim in the plan that `unsupportedChildRecipeRefusal` is what an unembedded recipe name meets)
- The cards are fully covered. Card 8:18 was the only card stating the wrong claim, and the fix rewrote it.
- Card 7:81 names `unsupportedChildRecipeRefusal` only for its admitted-recipe list, which is still correct.
- Cards 00, 03 and 06 mention `ProfileRecipe` but make no claim about the downstream refusal.

**LOW**: `_lym/plan/lens-review.md:15` still says the seed write's `unsupportedChildRecipeRefusal` "stays the refusal an unknown recipe meets". That is the same wrong claim F1 corrected in card 8. It is a disposition record from the plan-write lens review, not a card, so editing it may count as rewriting history. Either correct it or add a note that card 8 now says otherwise. The source lens reports (`lens-review.lens-way-forward.md:13`, `lens-review.lens-guard-reach.md:19`) are verbatim fork output and must stay as they are.

Tool calls:
1. Grepped `_lym/plan` for the refusal name and the Intent's phrases, to rerun F1's search over the whole plan. My context held only card 8.
2. Grepped `_lym` for files naming `unsupportedChildRecipeRefusal` or `ProfileRecipe`, to find sibling artifacts outside the cards.
3. Grepped cards 00, 03 and 06 for the matching lines, to check whether they repeat the claim. My context did not hold those cards.
