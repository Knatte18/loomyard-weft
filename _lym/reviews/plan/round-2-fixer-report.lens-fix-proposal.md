```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out (F1, F2, F3, F4, F6, F8, F9, and F7 in its adjusted form) by building a case where it is wrong.

**LOW: F8 leaves card 18's absent-parts-file promise at the CLI with no test.**
- Changed text: `_lym/plan/18-validate-parts-verbs.md`. `TestValidatePlanCmd_Part` no longer has "an absent parts file refused with its way forward".
- Case: line 39 still promises, at the CLI, "On a tapestry run that did not split, the parts file is absent, and the absent-parts-file refusal of `ValidatePart` answers."
  Line 40 names only one error branch: "an `ErrPartUnknown` error is refused with its text".
  An implementation could read that as `if errors.Is(err, loomshed.ErrPartUnknown) { refuse with envelope } else { return err }`.
  The absent-file error would then leave as a raw command error, without the envelope or its way forward, and the unknown-part row would still pass.
- So the reviewer's premise does not hold as the card is written. The unknown-part row pins the envelope of every `ValidatePart` error only if the card says every error from `ValidatePart` takes the same refusal.
- Card 7's `TestValidatePart` pins the error text, not the CLI envelope.
- Evidence: card 18 lines 39–40 as changed, and the review's F8 fix text.
- Sound alternative: keep the dropped row, or reword line 40 so every `ValidatePart` error is refused with its text, with `ErrPartUnknown` only one of them.

I found nothing wrong with the other proposals:
- **F1:** the spec ref now sits under Edit next to card 7's ImpactSummary, and the overlay run is clean.
- **F2:** `Part.Paths` comes from `Parts.External`, so "a target path the part's external exports hold" matches card 5's `resolvePass` bullet.
- **F3:** the three refs resolve and the rewording is comment-only.
- **F4:** with `--part` there are six modes and five flags, which agrees with card 18's "sixth mode exclusive with the four mode flags".
- **F6:** include resolution stays covered by seat-engine validation.
- **F7:** after the fixer's own correction, the resume end goes to card 10, whose line 37 holds it. The failed-gate and Accept-error ends share the `watch` predicate.
- **F9:** matches the template's blank lines at 73 and 77.

Further tool calls: none.
