```yaml
lens: fix-way-forward
outcome: reported
```

Task: check through the fix-way-forward lens that every refusal or next step the fix introduced can be carried out and leads to progress.

**MEDIUM** — D4, the new sentence "An ambiguous answer not split by build constraint is a `citation-symbol-ambiguous` finding naming the candidates' files … each with the way forward above."
- **Summary:** that way forward ("spell the symbol as a glyph") cannot be carried out for this case.
- **Evidence:** the token is either already a glyph, or a `pkg.Member` token D4 resolves as the glyph `<dir>#Member`.
  Spelling it as that glyph sends the same glyph to quarry and gets the same ambiguous answer.
  Quarry's `docs/glyph.md:265` defines `ambiguous` as several different declarations matching one glyph, such as Go build-tag duplicates.
  No respelling picks one of them.
  The writer's only ways past are a false ` (new)` or ` (removed)` mark, or dropping the backticks, and neither is named.
  The way forward should say what the writer does here, for example cite the declaring file by path, or mark the token as D3 says.

**LOW** — D5, the new refusal "a test reaching `NilSeams` whose markers miss a struct of the nested set the scan derives from the `Env` type".
- **Summary:** the refusal names no way forward, yet Constraints says "every new finding's text names its way forward, and a test reaches each one" (`PATTERN-refusal-way-forward`).
- **Evidence:** D5 does not say what the scan's message tells the writer to do.
  It should say to add a `//lym:seam <unit>#<Type>` line for each missing struct.
  The D5 refusals from earlier rounds have the same gap, but this one is new in this round.

The other new ways forward can be carried out:
- D6 "list it under Edit by its member glyph": `CheckEditTestFile` skips member glyphs.
- D6 Uses/Delete/Rename/Move file cover: avoids `edit-test-file`.
- D8 helper Bound "its card lists the `doc.go`": works through the Uses route.
- D4 rejected answer → `citation-symbol-missing`: the token can be corrected from the `lym quarry glyphs` listing.
- D4 field citation resolved by its prefix: a missing type yields a finding the glyphs listing can satisfy.
- D6 acceptance case for `landingshed.Deps`: reachable once the F1 markers name it.

Tool calls:
- Listed `shedrecipe.Env`'s field types and `envkit`'s `nilLegal`, to check whether D5's derived nested set holds a struct the writer could not mark validly (for example one from outside the worktree). My context showed only part of `Env`'s fields. I found none at the top level.
