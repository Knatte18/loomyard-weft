---
round: 1
focus:
  - "internal/commentlint/lint.go `carriesOver`: does the majority test (`2*unchanged > total`) match the decision record's rule, counting unchanged words across both lines? Check the tie and tiny-pair cases (one-word lines) against the record's Open risks and acceptance criteria."
  - "`newBaseBreaks` indexes base line pairs from every paragraph that `paragraphs` yields: can a pair from a paragraph kind the lint skips in the new text, or a pair that is no real wrapped break, let a genuinely new break through? Settle whether the base index and the new-text walk treat the same paragraph kinds."
  - "Order of checks in `findNewWrappedBreaks`: the record says the substitution match runs only after the word-pair lookup and the semantic-break check fail. Does the diff keep that, and does `carriesOver` still return true for every break the old `known[...]` lookup passed?"
  - "Word equality in `countEqualPositions` and `strings.Fields` on `content`: does it handle comment markers, empty lines and indentation the way `lineEndPair` does, so a pair is not compared on different text than the word-pair lookup uses?"
  - "internal/commentlint/lint_test.go: do the four new `TestFindNewWrappedBreaks` rows cover each acceptance criterion in the decision record, stay in the existing tier-0 table, and avoid duplicating an existing row? Is any behavior the diff adds left without a row?"
  - "internal/commentlint/doc.go: does the package doc describe the carry-over rule as built and agree with the code, per the repository's docs-land-in-the-same-commit rule? Also check scope: nothing outside the in-scope list (`diff.go`, the semantic-break rules, the gate) changed."
---

Seeded from the committed diff of `internal/commentlint` and the decision record's acceptance criteria.
