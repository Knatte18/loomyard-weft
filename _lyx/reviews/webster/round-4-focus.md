---
round: 4
focus:
  - "Sweep every Go comment hunk in the committed diff (git diff of the merge base with main to HEAD, all files) for an edited sentence whose lines break mid-phrase: PATTERN-comment-line-breaks allows a break inside a sentence only after a semicolon or a colon-ended part, or at a comma before an independent clause. Settle, for each file the diff touches under internal/planglyph, internal/planparser and internal/websterengine, whether any sentence the diff edited or added still has such a break, including a sentence whose first line is old text and whose tail is new. Check by reading the hunks, because lint-comments flags only breaks the diff itself creates."
  - "In internal/planglyph/deleteorder.go, read the doc comments on editTarget and collectEditTargets. Settle whether each parses as a sentence and whether the clause naming member glyphs and files attaches to the ref or target rather than to the card."
  - "In internal/planglyph/doc.go, read the sentence naming the passes that raise glyph-rejected. Settle whether it names every pass in the package that raises that finding, or names a group instead of enumerating files."
  - "In internal/planglyph/glyphchain_test.go, read TestGlyphChain_CrossCard. Settle whether every field of its case table is set by at least one case or read by the assertion."
---

Round 3 closed with the comment-wrap defect still open on new sites, so the first entry asks for a sweep of the whole diff instead of a re-check of known sites.
