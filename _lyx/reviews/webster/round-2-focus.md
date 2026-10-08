---
round: 2
focus:
  - "Comment line breaks: in the Go comments the branch's commits add or edit (for example in internal/planglyph/doc.go, repo.go, planglyph.go, handle.go and internal/websterengine/beginbatch.go), does any edited sentence still break mid-phrase, where PATTERN-comment-line-breaks allows a break only at a sentence end, after a semicolon or at a comma before a coordinating conjunction? Settle it by reading the changed comments, since lyx loom lint-comments covers only part of the rule."
  - "Glyph chokepoint in the redundancy check: in internal/planparser/redundancy.go, is every key stored in or looked up from the containers map (member glyphs in redundancyOwnerOf, package and file self glyphs, and the rename source in renameToSideUnits) produced by Glyph.UnitPath, never by reading Glyph.Unit as a disk path, as PATTERN-glyph-conversion-chokepoint requires?"
  - "Repeated targets: when one card lists the same ref twice, does planGatePass in internal/planglyph/plangate.go emit one finding per file and member, and does coverageSubjects in internal/planglyph/callercoverage.go emit one caller-uncovered finding per subject and file, as the plan's cards 8 and 11 state? Is there a test row for each?"
  - "Interface-method detection: do isInterfaceMethod in internal/planglyph/plangate.go and the matching test in internal/planglyph/handle.go classify an interface method whose name begins with func correctly, and does renameDeclSource hand quarry.Name the right spec for it?"
  - "Markdown line breaks: in the spec and docs the branch edits (contracts/specs/loom-plan-spec.md, the rewritten handle-checks paragraph and the edited glyph-malformed row), does each edited sentence sit on its own line, with table cells left on one line?"
  - "Package overview accuracy: does the overview in internal/planglyph/doc.go state which of Validate, ValidateFormat and ValidateDispatch append planGatePass, matching the callers in the code?"
---

The entries name where to look and which question to settle; the reviewer re-reads the diff for each.
