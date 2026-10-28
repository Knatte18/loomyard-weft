```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of the record's own decisions that act on the same state, and checked them against the existing plan checks that work on the same card refs.

**Findings**

1. **BLOCKING, D8 (also D6): the `Uses` way forward collides with `uses-later-target`.**
   - D8 says a `doc.go` listed under `Uses` "adds the dependency edge … which orders the card after a card that edits that doc".
   - That is false. Plan spec check 23, `uses-later-target`, refuses a card's `Uses` entry that a higher-numbered card targets. It does not reorder the cards.
   - The common plan shape breaks: several cards edit code in a package, and a last card edits its `doc.go`. Each earlier card that follows D8's "else under Uses" advice now fails `uses-later-target`.
   - D6 has the same problem when a later card edits the seam test.
   - The decision has to say what the writer does here: list the doc under Edit, order the doc card first, or have D8 accept a later card's Edit as covering it. The edge claim has to go.

2. **MEDIUM, D8 and D6: listing the file conflicts with `redundant-package-target`.**
   - A card whose target is a package self glyph (`internal/foo#`) and which follows D8's "under Edit" advice for `internal/foo/doc.go` trips `redundant-package-target` (spec check 24). That check refuses a file self glyph next to the package self glyph of its own package.
   - D6 hits the same check when the seam test sits in the package the card already targets as a whole.
   - D8 and D6 need to say that a package self glyph target already counts as listing the doc or test.

3. **MEDIUM, D8: a card that deletes, renames or moves a whole package can't pass.**
   - D8 accepts `doc.go` only under Edit, Extends or Uses.
   - A card that deletes or moves package P, `doc.go` included, carries `doc.go` under Delete or Move.
   - Listing it a second time under Uses trips `card-field-overlap`. Listing it under Edit makes the same ref appear under two type labels.
   - D8 should accept a `doc.go` that the card's own Delete, Rename or Move group already names.

4. **MEDIUM, D2 against D3 and D4: acceptance criteria name things that don't exist on purpose.**
   - Acceptance criteria often describe negative scenarios with names that must not exist. This record's own D4 criterion cites `loomshed.NoSuchFunc` and `internal/loomshed#NoSuchFunc`.
   - D3 and D4 test every writer-authored section, so such lines fail with `citation-symbol-missing`.
   - The only escape is a ` (new)` or ` (removed)` mark, which misdescribes them.
   - The record needs either an exemption for the Acceptance criteria section, or an example-name rule in the spec.

5. **LOW, D4 against D3: the order of the ambiguity test and the mark is unstated.**
   - D3 says a marked token "is not tested". D4 raises `citation-symbol-ambiguous` from the package-directory mapping.
   - The record doesn't say whether a ` (new)`-marked `pkg.Member` whose `pkg` two directories share is still refused as ambiguous.
   - That matters for a new symbol in a package whose base name another directory also has.

6. **LOW, D5 against D6 and D9: where the marker parser lives.**
   - D5 puts the marker parser in `internal/impactset`, which imports `gitexec`, `gitkit` and `logger` (and the `tmuxkit` test kit).
   - D9 lets D6 live in `planparser` or `planglyph`. `planparser` is the pure layer, so importing `impactset` from it would be wrong.
   - The record should pin D6 to `planglyph` (or the loomshed layer), not "whichever hosts".

**Pairs checked**
- D1 and D2: both read the `D<n>` headings. Held.
- D1 and D2 against `lym loom decision add` (which runs `Validate` only) and the Added-after-Discussion entries: held.
- D1 against carry-over blocks: they live under Open risks, not Decisions. Held.
- D2 against D3 and D4: the checks overlap on the acceptance lines. Broken (finding 4).
- D3 against D4: glyph tokens drop out of the path rule because of `#`, and path tokens don't fit the `pkg.Member` shape. Held. The mark-versus-ambiguity order is unstated (finding 5).
- D3 and D4 against D2's darn split: the darn check runs D1 and D2 only. Held.
- D3 and D4 against the exemptions for Added-after-Discussion entries and carry-over blocks: held.
- D4 against the quarry-outage error path in the Discussion-Write and Discussion-Burler-Fix gates: held. An error halts the run and never passes it.
- D5 against D6: the marker set feeds the check. Held. Parser placement is open (finding 6).
- D5 against D7: the new scan's test file needs a stated tier. Held.
- D6 against D8: both add `Uses` listings. Broken through `uses-later-target` and `redundant-package-target` (findings 1 and 2).
- D8 against Delete, Rename and Move groups: broken (finding 3).
- D6, D7 and D8 against D9: done cards skipped, `language: go` only, not run at dispatch. Held.
- D10 against the `PATTERN-stencil-ownership` constraint: held.
- D11 against the PATTERN change and D5: held.
- Acceptance criteria for D6 and D8 ("listing it under Uses passes"): these pass only when no later card edits the listed file. Broken (finding 1).

**Tool calls**
- Read the plan spec text for `uses-later-target` and `redundant-package-target`. My context held the check names but not their rules.
- Read `internal/impactset`'s imports and searched `planglyph` and `planparser` for an existing `impactset` import. My context didn't show the package's dependencies.
