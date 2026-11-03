```yaml
lens: code-tests
outcome: reported
```

**Tests per behaviour**

- **Single bare reference moves with its card.**
  - Pinned: `TestRenumber` (`gammaIntent` "card 2" → "card 3" in the insert rows, `renumber_test.go:80-82`); `TestRenumber_RewritesCardIDs` ("card 9" → "card 10"); `TestRenumberVerbs` insert row.
- **Comma list moves.**
  - Pinned: `TestRenumber`, "cards 3, 4" and "cards 2, 3", `renumber_test.go:80-82`.
- **Range starting at the insert position moves.**
  - Pinned: `TestRenumber` insert rows, "Covers cards 2–3" → "3–4". The range starts at the insert position, so the `e.position <= number` clause does not fire.
- **Remove: a range starting at the removed card is kept and its line reported.**
  - Pinned: `removeSecondMentions`, `renumber_test.go:94`.
- **Remove: a single reference to the removed card is kept and reported.**
  - Pinned: `removeSecondMentions`, `renumber_test.go:96`; `TestRenumber_RewritesCardIDs` Remove; `TestRenumberVerbs` remove row.
- **Remove: a reference to the removed last card is reported when no card moves.**
  - Pinned: `removeLastMentions`, `renumber_test.go:102`.
- **Card id is masked from the bare scan.**
  - Pinned: `TestRenumber_RewritesCardIDs`, `--card 10-beta` and `9-beta` stay ids and are reported once.
- **`rewritten_refs` and `prose_mentions` in the envelope.**
  - Pinned: `TestRenumberVerbs`, `loomcli/renumber_test.go`.
- **Bare reference in a card that keeps its number is rewritten.**
  - Pinned for the CLI insert only: `TestRenumberVerbs`, alpha "card 2" → "card 3". The remove and `TestRenumber` rows have no unmoved card holding a bare reference.
- **Glyphless admit.**
  - Pinned: `TestCheckEditTestFile` ("test file with no member glyph", "another test file"); `TestIndex_AppendsPlanChecks` (format and rework with `var _ = 1`, plus the `TestX` refusal row).

**Findings**

1. **MEDIUM, design (coverage gap).** No row covers a range the inserted or removed card falls strictly inside, or a range ending at the insert position. The kept-range branch `inRange && previous < e.position && e.position <= number` in `mapsFaithfully` is never reached on insert. Nothing in the diff exercises `cardMentionRe`'s hyphen, em dash, "and" list or `Card` (capital) forms either. The only kept-range row is the remove case where the range starts at the removed card, which the `number == e.position` clause catches. Deleting the range clause would not fail any test. Location: `renumber_test.go:46-55` (fixture) and `renumber.go:729`. Fix: add a fixture line and a row, such as "cards 1–3" with a card inserted at 2, and "cards 1–2" with an insert at 3 as the boundary.
2. **LOW, design.** No test pins that a card file whose only change is a bare reference is journaled and restored on failure. `TestRenumber_CardIDRewriteFailureReversed` faults `08-alpha.md`, which changes by card id and not only by bare reference. The `len(refs) == 0` skip and the `stayed` write path for a bare-only change are pinned only by the CLI insert row in `TestRenumberVerbs`.
3. **LOW, design.** No test pins that a bare `card N` on a Card Index line, or on a heading line, is skipped (`cardIndexLines`, `headingLine`). A regression that rewrote index lines twice would pass.
4. **LOW, design.** `glyphlessFile`'s failure paths have no row. These are a toc error, a lossy file, an error entry, nil Symbols, and a file missing from the toc (the "file does not exist yet" case the summary claims is still refused). `TestIndex_AppendsPlanChecks` pins only the empty-symbols admit and the member refusal. The existing "test file under Edit" rows run against an empty `t.TempDir()`, which only reaches the toc-error or missing-file path through the quarry error and so may cover it incidentally. The `path.Base` match against a same-base file elsewhere is also unpinned.

**Test economy.** I found no redundant test. The new rows extend the existing tables, so no duplicate-coverage finding.

**Tool calls (1)**
- `grep` of `renumber_test.go` and `index_test.go`: the manifest held only the diff hunks, and I needed the fixture lines to place each row.
