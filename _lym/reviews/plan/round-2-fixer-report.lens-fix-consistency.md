```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the card 8 fix agrees with the sibling cards, rows and cross-references it touches.

1. **LOW**: `_lym/plan/lens-review.md:15` still repeats the claim the fix removed from card 8.
   - **Concerns:** card 8's Intent sentence, which the fix rewrote; this line is its twin elsewhere in the plan directory.
   - **Evidence:** the line says that keeping the recipe means "the seed write's `unsupportedChildRecipeRefusal` stays the refusal an unknown recipe meets". The rewritten card 8 says an unknown name is refused through `shedrun.ValidateRecipe` and `battenshed.ErrUnknownRecipe` (`internal/battencli/wire.go:1392`), before `unsupportedChildRecipeRefusal` at `:1400` is reached.
   - **Caveat:** this file records an earlier lens review, so it may count as history rather than live plan text. If the fixer treats it as plan text, it should correct or annotate the clause. The lens fork reports behind it (`lens-review.lens-way-forward.md:13`, `lens-review.lens-guard-reach.md:19`) are verbatim records and must not be edited.

The other references agree with the change:
- Card 8's third `TestPrepareRunSettings` row ("writes the task's settings alone, with no refusal") matches the new "passes `prepareRunSettings` with its settings alone".
- Card 7 (`07-arras-seedable.md:12`, `:81`) only changes which recipes `unsupportedChildRecipeRefusal` admits, and the fix no longer depends on that.
- Card 3 (`:21`), card 6 (`:75`) and the overview (`:23`) say nothing about the refusal path.

Tool calls:
- One grep of `_lym/plan` (and `_lym/discussion*`) for `unsupportedChildRecipeRefusal`, `ProfileRecipe`, "admitted recipes" and "run-settings refusal". My context showed only card 8, not its siblings or the plan's lens records.
