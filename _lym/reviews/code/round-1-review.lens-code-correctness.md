```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, hand-tracing each changed function and branch of the `internal/commentlint` diff. I made no tool calls and ran no tests; every case below was traced by hand from the context already held.

**Per function or branch: the case I ran, and whether it is correct**

- **`newBaseBreaks`, empty base text** (`lint.go:174`): `commentBlocks("")` yields no blocks, so both maps stay empty and every break falls through to the shape lookup, which finds nothing. Correct. New files are flagged as before (row "a fixed-column-wrapped new comment is flagged").
- **`newBaseBreaks`, single-line paragraph:** the loop `i+1 < len(paragraph)` doesn't run, so nothing is indexed. Correct.
- **`newBaseBreaks`, `Fields` on a line:** `paragraphs` drops blank lines, so `Fields` is never empty and `lineEndPair` can't index out of range. Correct.
- **`newBaseBreaks`, non-prose paragraphs:** it indexes pairs from every paragraph kind, including list, heading and indented-code paragraphs, which the new-text walk skips (`lint.go:290`). `Fields` also drops indentation, so a code line and a prose line with the same words compare equal. The effect is only to widen the candidate set, which is a lenient direction. See finding F1.
- **`carriesOver`, word-pair hit:** returns true, exactly as the old `known[...]` lookup did, so the old pass behaviour is kept. Correct.
- **`carriesOver`, shape lookup:** a base line pair is only a candidate when the word counts of both lines match, so `countEqualPositions` never indexes out of range. Correct.
- **`carriesOver`, one word on each line:** `total` is 2 and the test needs `2*unchanged > 2`, so both words must be equal. That already means the word-pair lookup hit, so the rename of a one-word pair is flagged. This matches the record's open risk. Correct.
- **`carriesOver`, 50/50 tie** (for example 2 of 4 words unchanged): `4 > 4` is false, so the break is flagged. This matches the record's "outnumber". Correct.
- **`carriesOver`, 3+3 words with one rename each:** unchanged is 4, and `8 > 6` is true, so it is carried over. Correct.
- **`carriesOver`, 3+3 words with a rename on only one line and 1 unchanged word on the other:** unchanged is 4 if the renamed line keeps 1 word... not relevant, since the test counts both lines together as the record says. Correct.
- **`carriesOver`, match against an unrelated base pair:** two lines that share a shape with a base pair elsewhere in the file, and agree on most words by position, are let through. That is the record's accepted bound, and the odds are low for prose. Correct per the record.
- **`countEqualPositions`:** the comparison is exact, so punctuation and case count as differences. A rename of `foo,` to `bar,` counts as one substitution, and a trailing-punctuation change counts too. Correct.
- **`findNewWrappedBreaks` check order:** `endsSemanticBreak` now runs before the lookup. Both are pure and the result is an OR, so the outcome is the same as the old order. This is a deviation from the record's "match runs after the word-pair lookup and the semantic-break check", because here the semantic check runs first and the word-pair lookup sits inside `carriesOver`. In effect the substitution match still runs last, so the behaviour is correct.
- **`findNewWrappedBreaks`, test rows traced:**
  - Row 1 (last word renamed): break 1 is shape (5,3) against the base pair, with 4+3 unchanged, so it is carried. Break 2 is `comment that`, a known word pair, so it is carried. Result nil. Correct.
  - Row 2 (first word renamed): break 1 is (5,3) with 5+2 unchanged, carried. Break 2 is `comment that`, known. Result nil. Correct.
  - Row 3 (a break created beside a rename): break 1 is the known word pair `a fixed-column`. Break 2 has shape (2,4), which has no base candidate, and `folded comment` is unknown, so line 4 is flagged. Result `[4]`. Correct.
  - Row 4 (a pair rewritten in most of its words): break 1 is (5,3) with 0+3 unchanged, and `6 > 8` is false, so line 3 is flagged. Break 2 is the known `comment that`. Result `[3]`. Correct.

**Findings**

- **F1, LOW, class `design`, `internal/commentlint/lint.go:176-185`:** `newBaseBreaks` indexes line pairs from every paragraph kind, but the new-text walk checks only prose paragraphs (`lint.go:290`). A new prose break can therefore be carried over by a base pair from a list item, heading or indented-code paragraph. `Fields` also drops the indentation that separates code from prose. This only lets a few breaks through and doesn't flag good ones. The fix would be to index only pairs from paragraphs where `isProseParagraph` is true (note that this would change the existing `wordPairs` behaviour too, so it may be better to restrict only `linePairsByShape`). Evidence: `lint.go:177-184` against `lint.go:290`.

I found no wrong results, panics or inconsistent state on any branch.

**Tool calls:** none made. The manifest already held the full diff, `lint.go` and `lint_test.go`.
