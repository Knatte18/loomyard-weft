```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 fix set agrees with the sibling decisions, criteria and cross-references it touches (fix-consistency lens).

- **MEDIUM**: the D6 rule 4 change contradicts the D6 criterion it left untouched.
  - Changed text (rule 4): "A failure removes the sibling and leaves `<dir>` as it was, except a failed rename after that removal, which leaves `<dir>` absent".
  - Untouched criterion: "D6: an extract that fails part-way leaves `<dir>` as it was, absent or empty, and no sibling behind."
  - The criterion still promises `<dir>` "as it was" on every failure. Under rule 4, a `<dir>` that was empty before the extract can end up absent after a failed rename. A plan writer following the criterion would test for a state that rule 4 says won't hold.
  - Evidence: D6 rule 4 against the D6 acceptance criterion for an extract that fails part-way, both in `_lym/discussion/decision-record.md`.
- **LOW**: the D6 bound sentence now understates what rule 4 does.
  - Untouched text: "writes only under `<dir>` and a temporary sibling of it".
  - Rule 4 now also removes an existing empty `<dir>` itself. That removal is outside "under `<dir>`", and the bound sentence doesn't mention it.
  - Evidence: the D6 "Bound:" paragraph against rule 4 in `_lym/discussion/decision-record.md`.

The other twins agree with the changes:
- **D5:** the hand-merge message sentence, "the row's landing commit names it", is now true. The resume criterion, the Notes entry and the "once the landing commit is final" ordering all agree.
- **D3:** the rationale, the equality criterion and the seed criterion all agree.
- **D6:** the refusal criterion now includes the sibling refusal.

Further tool calls: none. My context held everything this lens needed.
