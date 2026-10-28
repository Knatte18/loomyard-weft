```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, one pass over the tests the diff adds against the behaviours it adds. `wrapped` is the base text in `lint_test.go`.

**Behaviour → pinning test**

| Behaviour | Pinned by |
|---|---|
| Last-word rename at a break is not flagged. Word-pair miss, shape (5,3) base pair, 7 of 8 words unchanged. | Row "a rename of a legacy wrapped line's last word is not flagged" (`lint_test.go:59`). It fails if `carriesOver`'s substitution branch is removed. |
| First-word rename at a break is not flagged. | Row "...first word is not flagged" (`lint_test.go:65`). The rename lands on the line-start word of the next line. |
| Unchanged words count over both lines. | The two rename rows above together. Counting only the before-line fails the last-word row (4 of 8 is not a majority). Counting only the after-line fails the first-word row (2 of 3 words). |
| A break created beside a rename is still flagged. | Row "a break an edit creates beside a rename..." (`lint_test.go:71`). It flags line 4 because no base pair has shape (2,4), so the word-count gate is what it exercises. |
| A line pair rewritten in most of its words is flagged. | Row "...rewritten in most of its words is flagged" (`lint_test.go:77`). 3 of 8 words are unchanged, which fails the majority test. |
| Existing rows keep their results. | The unchanged existing rows. |
| Word-pair lookup still passes first. | The existing "untouched wrapped block" rows. |
| Semantic-break check still passes. | The existing sentence-end, comma and semicolon rows. |
| Shape-indexed lookup (`linePairsByShape`) and the same-word-count requirement. | Row at line 71, indirectly. It never reaches `countEqualPositions`. |
| Doc claim: a word-count-changing rename still flags. | Row at line 71 (`wrapped` → `folded` plus a reflow). |

**Findings**

1. **LOW, class `design`**, `internal/commentlint/lint_test.go:77-82` (code at `lint.go:200`). The majority boundary is unpinned. The only threshold row has 3 of 8 unchanged, so replacing `2*unchanged > total` with `>=` still passes every row. The decision record names the tie and tiny-pair cases (Open risks: two lines of two words each with a rename on each line, 2 of 4, is still flagged). No row covers an exactly-half pair. Fix: add a row where a pair has exactly half its words unchanged, with want flagged, for example base `a b`/`c d` against new `a x`/`y d`.

2. **LOW, class `scope`**, `internal/commentlint/lint_test.go` (code at `lint.go:174-187`). The base index takes line pairs from every paragraph, including list, code and heading paragraphs that the new-text walk skips, and from any position in the file. No row covers a pair that matches only a non-prose or unrelated base paragraph. The behaviour is untested, though it has been implemented. Fix: add a row where the base holds a list or indented paragraph pair that differs from a new wrapped pair by one word, then settle the intended result.

3. **NIT, class `scope`**, `internal/commentlint/lint_test.go:65-70`. Rename at both sides of one break at once (line-end word and next line-start word renamed together) has no row. The word-pair miss and the substitution path are only exercised for a single renamed word per break. Fix: optionally add one row with both words renamed.

The line-71 row never reaches the substitution path (shape miss), so it pins the shape gate, not the "unchanged words are a minority" rule. That is acceptable for its stated acceptance criterion but weaker than its name suggests.

**Tool calls:** none. The manifest already held the diff, `lint.go` and `lint_test.go`.
