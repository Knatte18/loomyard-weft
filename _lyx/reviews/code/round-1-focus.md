---
round: 1
focus:
  - "internal/discussionparser/patternchanges.go `patternOverCapFindings`: does the measured line (`- \x60<ID>\x60 — <text>`, runes) match what `internal/planparser/patternentry.go` `pattern-entry-line-cap` measures for the same bullet, including the text being trimmed and the bullet's appended-after-Discussion suffix being excluded?"
  - "`Validate` in validate.go and `AppendDecision`: does an over-cap `PatternChange` reach `patternOverCapFindings` on the appended bullet, and is the record restored byte-for-byte on that finding?"
  - "Tests in validate_test.go and append_test.go: do they cover the exactly-at-cap pass, one rune over, multi-byte runes counted as runes, a `removes` bullet never measured, the appended-bullet way forward, and the cap-equals-`pattern.MaxEntryLineChars` pin, without duplicating one another?"
  - "Docs landing in the same diff: `contracts/specs/loom-discussion-spec.md` states the cap and the `internal/discussionparser/doc.go` text names the check; do the new comments follow the repository's comment conventions and the CLAUDE.md prose rules?"
  - "Scope against the decision record: `ParsePatternChanges` stays a shape check, and the diff leaves the plan gate, the PATTERN format checker and the cap value untouched."
---

Seed focus for round 1, set from the board entry, the decision record and the diff stat.
