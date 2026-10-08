---
round: 3
focus:
  - "Comment line breaks: in internal/planglyph/planglyph.go (the ValidateDispatch doc), internal/planparser/handle.go (file header), internal/planparser/handle_test.go (file header), internal/planparser/shape.go (the ledger doc) and internal/planparser/validate.go (the checkGlyphMalformed doc), does every sentence the branch's commits edited sit on one line or break only at a semicolon or a comma before an independent clause, per PATTERN-comment-line-breaks? Check by reading the sentences, since lint-comments flags only breaks the diff creates."
  - "Comment line breaks elsewhere: across every Go comment the branch's diff touches, is any sentence the diff edited (not merely a neighbour) still broken mid-phrase by an old column wrap? Name the file, line and sentence."
  - "internal/planglyph/deleteorder.go, the editTarget type: does its doc comment parse as a sentence and state what the type holds, an Edit-group ref of a card with its shape and disk path?"
---

The focus targets the one gating defect class still open and its sibling sites.
