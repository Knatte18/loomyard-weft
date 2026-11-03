```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over `planparser/renumber.go`, `planglyph/index.go` `glyphlessFile`, and `planparser/edittestfile.go`. I traced inputs by hand and found no defects.

**Per function, the case run (correct or not):**

- `mapsFaithfully`, insert at p=4:
  - "cards 3–5" is kept (3<4≤5). Correct.
  - "cards 3–4" is kept, because the new card lands inside it. Correct and conservative.
  - "cards 4–6" is rewritten to "5–7", since it starts at p and the new card is not in it. Correct.
  - "cards 2–3" is untouched. Correct.
  - "cards 3, 5" rewrites 5 only, because the list is not a range. Correct.
- `mapsFaithfully`, remove at p=4:
  - A reference naming 4 is kept, including "cards 3–4" and "cards 4–6". Correct.
  - "cards 2–5" is kept, because the range would lose a card. Correct.
  - "cards 5–6" becomes "4–5". Correct.
  - "cards 1–2" is untouched. Correct.
- `rewriteCardMentions`:
  - "card 9" becomes "card 10", with `from`/`to` recorded. Correct.
  - A number not in `moves` (unmoved, or not a card) stays, and the `to == ref` check skips the pair. Correct.
  - A kept reference is left whole while another reference on the same line is still rewritten. Correct.
  - Dash tokens fail `Atoi` and are carried through by `ref[at:tok[0]]`. Correct.
  - Spans are computed on the masked line and applied to the original line. This is safe because the mask preserves length.
- `maskCardIDs` and `cardIDTokens`: "card 9-beta" and "--card 9-beta" are masked, so no bare match forms. The bare rewrite does not run on those spans. `rewriteCardIDs` then moves the id once. Correct.
- `renumberReferences`:
  - `keptAt` indexes are stable because the id rewrite never changes the line count.
  - The mention text is read after the id rewrite. Correct.
  - Refs sort bare-then-id, then stable by file and line, matching the tests.
- `applyRenumber`, a card that stays in place:
  - It is journaled and written only when `len(refs) > 0`. Its `kept` mentions are collected before the skip, so the report stays complete.
  - A rewritten in-place card is in `journal.Files`, so `reverseRenumber` restores it after a failed write.
  - Remove-last (empty `moves`) still reports the mention for the removed card.
  - Insert at last+1 (empty `moves`) is a no-op for refs. Correct.
- `applyRenumber`, a moved card: line 0 (the heading) is skipped, and `moveCardBytes` has already updated it. Correct.
- `cardIndexLines`: the overview-line shift is the number of newlines in the frontmatter prefix, so the skip covers exactly `lines[headingIdx+1:end]`. The predicate is the same as before. I compared it with the removed `proseMentions` at `486bbf44:internal/planparser/renumber.go:675-690` and found the same skip rules: card files skip line 0, and the overview skips Card Index lines. Correct.
- `glyphlessFile`:
  - A TOC error returns false, so the refusal stands.
  - `Error != ""` or `Lossy` returns false.
  - `Symbols == nil` ("not requested") returns false. `Symbols` present and empty returns true.
  - A missing file returns an error or an empty `Files`, so false.
  - A file target gives a one-entry `DirAnswer` in quarry's `DirAnswer` doc, so the `path.Base` match cannot pick up another file's entry.
  - `ValidateFormat` and `ValidateRework` both pass `worktreeRoot`. Correct.
- `CheckEditTestFile`:
  - `glyphless` is evaluated last in the `||` chain, so it runs only for a path the check would otherwise refuse.
  - The only callers are the two in `index.go`, and both pass a non-nil function, so a nil `glyphless` cannot reach the call.
  - A non-matching path from the callback still produces the finding. Correct.

**Findings:** none.

**Observation, not a finding:** a zero-padded bare number such as "card 09" is rewritten without padding ("card 10"). I judge this harmless, because the card-id form carries the padding and bare prose normally does not.

**Tool calls:**
1. `git show 486bbf44:internal/planparser/renumber.go | grep -n "func proseMentions" -A 45` — my context lacked the removed `proseMentions` body, which I needed to compare skip semantics.
