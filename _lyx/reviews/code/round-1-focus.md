---
round: 1
focus:
  - "internal/pattern/check.go `EntryRuleText` vs internal/discussionparser/patternchanges.go `patternRuleText`: do the two extractions cut the same text on lines with and without `(test)`, with and without a link, with both, and with a name-less line, and does the pinning test cover those rows?"
  - "internal/discussionparser: does the bullet text `lyx loom decision add --pattern-text` appends reach `patternOverCapFindings` through the same record check, so an over-cap text is refused at record time, and is a test covering that refusal path?"
  - "Stale references: does any doc, spec, stencil, test or message still describe the cap as whole-line or name `MaxEntryLineChars`, 360 or `patternEntryLineCap`, in contracts/specs, the plan stencil, doc.go files and PATTERN.md or pattern/?"
  - "internal/planparser/patternentry.go: does a prescribed line pass when only its name and link push the whole line past the old cap, and fail when its rule text exceeds 280, as the acceptance criteria say?"
  - "Test economy: do the new and changed tests in check_test.go, planparser and discussionparser validate_test.go add rows to existing tests rather than duplicate coverage?"
---

Seed focus from the board entry, the decision record's acceptance criteria and the diff stat.
