---
round: 2
focus:
  - "internal/planglyph/resign.go: does any call outside internal/planparser pass a glyph.Language (such as glyph.CSharp) to glyph.Parse, given the declared PATTERN-glyph-conversion-chokepoint text that a plan ref's or handle's alphabet is decided only via planparser?"
  - "PATTERN.md and pattern/ diff: does it change only PATTERN-glyph-conversion-chokepoint, with PATTERN-cwd-resolution and PATTERN-sparse-hub-base keeping their original wording?"
  - "internal/planglyph/languagemismatch.go and csharphead.go: does a .cs path target beside an unresolved Go member glyph still raise plan-language-mismatch, and does renameCSharpDecl refuse an instance constructor head, with test rows pinning each?"
---

Focus entries name where to look; the review range is unchanged.
