# Discussion check refuses a PATTERN text over the entry line cap

A decision record's `## PATTERN changes` bullet could declare an `adds` or `changes` text whose PATTERN.md entry line ran past `pattern.MaxEntryLineChars`.
The discussion check passed it, and the run first failed at Plan-Write.
There, `pattern-entry-line-cap` refuses the over-cap fenced line, while `pattern-changes-mismatch` and `pattern-line-unprescribed` refuse any other text, so only the parent replacing the bullet could unblock the run.

## What changes

- `discussionparser.Validate` gains a `pattern-change-over-cap` finding.
  It runs after `ParsePatternChanges`, once per `adds` or `changes` bullet whose overview line, `` - `<ID>` — <text> ``, is longer than the cap in runes, the same line and unit the plan gate measures.
  The detail names the entry, the length and the cap, and its way forward says to shorten the text or move the detail to the entry's background file.
- A bullet appended by `lyx loom decision add` is measured too, and its way forward names re-recording the entry through that verb, which replaces the bullet in place.
- Because `AppendDecision` runs `Validate` after its write and restores the record on a finding, `lyx loom decision add --pattern-text` now refuses an over-cap text with the record unchanged.
- `contracts/specs/loom-discussion-spec.md` states the cap in its PATTERN changes section, so Discussion-Write shortens the text before its review, and the package doc names the check.

## Design decisions

- The length check stays out of `ParsePatternChanges`.
  `DeclaredPatternChanges`, the plan gate's mismatch check and the dropped-bullet check all refuse on any parse finding, so a shape finding would turn an over-cap bullet into hard errors at later steps.
  As a `Validate` finding it reaches every discussion gate, `lyx loom validate-discussion` and `AppendDecision` through one site.
- `internal/discussionparser` declares its own unexported `patternEntryLineCap`, because its leaf test admits no non-stdlib production import.
  An in-package test pins it equal to `pattern.MaxEntryLineChars`, so a change to either cap fails that test.

## Tests

`TestValidate_PatternChangeOverCap` covers `adds`, `changes`, appended and multi-byte bullets one rune over the cap, the same at the cap, and a `removes` bullet that would be over the cap if it were measured.
`TestAppendDecision_FindingRestoresRecord` gains a row for an over-cap `PatternChange`.
