---
round: 1
focus:
  - "internal/planglyph/donecheck.go: check the counterpart match is gated on a Go plan and applies to Create targets and Rename new sides, and that a member declared in an unrelated unit still yields create-not-done."
  - "internal/planglyph/handle.go: check a handle binds through the counterpart only when its expected glyph matches no delta symbol, binds to the counterpart glyph, and cannot bind ambiguously or double-bind a delta symbol (bind-count-mismatch)."
  - "internal/planparser/targetdirs.go: check the exported helper maps both directions (<dir> and <dir>_test), goes through ParseRef and Glyph.String, and handles non-Go or malformed glyphs and a directory whose own name ends in _test."
  - "Tests: check the new cases are rows in the existing done-check, binding and targetdirs tests, and that each acceptance criterion in the decision record (both directions, unrelated unit, handle binding, C# no lookup) has coverage."
  - "Docs: check planglyph doc.go and pattern/PATTERN-glyph-conversion-chokepoint.md name the helper accurately and match the behavior the diff implements."
---

Seed focus drawn from the board entry and decision record scope.
