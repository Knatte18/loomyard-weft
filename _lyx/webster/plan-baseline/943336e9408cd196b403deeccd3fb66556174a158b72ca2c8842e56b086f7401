# Card 6 — discussion-append-pattern-change

**Create:**
- `plan:internal/discussionparser#patternChangesInsertOffset` -> `func patternChangesInsertOffset(content string) (offset int, needHeading bool)`
- `plan:internal/discussionparser#checkPatternChangeDropped` -> `const checkPatternChangeDropped = "pattern-change-dropped"`

**Edit:**
- `internal/discussionparser#AddedDecision`
- `internal/discussionparser#AppendDecision`
- `internal/discussionparser#CheckAddedDecisionsKept`
- `internal/discussionparser#TestAppendDecision_AppendsAtEndOfDecisionsSection`
- `internal/discussionparser#TestAppendDecision_FindingRestoresRecord`
- `internal/discussionparser#TestCheckAddedDecisionsKept`
- `internal/discussionparser/doc.go`

**Uses:**
- `plan:internal/discussionparser#ParsePatternChanges`
- `plan:internal/discussionparser#PatternChange`
- `internal/discussionparser#fencedLines`
- `internal/discussionparser#addedDecision`
- `internal/discussionparser#writeFixture`
- `internal/discussionparser#allSectionsContent`

**Intent:**
D5.2's parser half: `AppendDecision` can append one `## PATTERN changes` bullet beside its `### Added after Discussion` entry, in the same write, and the drop check covers the bullet.

- `AddedDecision` gains `PatternChange *PatternChange`; nil keeps today's behavior byte for byte.
- With it set, `AppendDecision` trims its `ID`, `Action` and `Text`, writes the decision entry exactly as today, and writes the bullet `PatternChange.Bullet()` followed by ` — added after Discussion (<by>, <date>)`, the date the UTC day of `d.Date`, the same day the heading carries.
  `patternChangesInsertOffset` places the bullet after the section's last non-blank line when the section exists, and otherwise reports that a `## PATTERN changes` heading must be created: just before `## Notes for the plan writer` when the record has that heading outside a fence, else at the end of the record.
  The record is written once with both insertions, then `Validate` runs as today, so a bullet of a wrong shape (an unknown action, `adds` without text) restores the prior bytes and returns the findings with a nil error; nothing else in the record changes.
  It removes and rewrites nothing.
- `CheckAddedDecisionsKept` also returns one `checkPatternChangeDropped` finding per appended bullet, keyed by entry ID, author and date, that some version holds and the record does not, in order of first appearance, its detail naming the bullet and saying to restore it from the record's history, since an appended bullet is never the writer's to remove.
  A version whose section does not parse contributes no bullets, as for headings; a record whose section does not parse yields no drop finding of its own, since `Validate` reports it.
  The check runs where the dropped-entry check for `### Added after Discussion` headings already runs, the darn discussion gate and the verb's darn-record check, as D5.2's "as for `Added after Discussion` entries" sets; a loom run's discussion check reads no history for either.
- `doc.go` says `AppendDecision` also appends a PATTERN change.

Tests:
`TestAppendDecision_AppendsAtEndOfDecisionsSection` gains rows whose decision carries a `PatternChange`: a record without the section gains it just before `## Notes for the plan writer`; a record without either heading gains it at the end; a record with the section gains the bullet after its last bullet; each row asserts the whole resulting record, the decision entry included, and the support log untouched, as the test's rows already do.
`TestAppendDecision_FindingRestoresRecord` gains a row whose `PatternChange` carries an unknown action: the record stays byte-identical and a `pattern-change-malformed` finding comes back.
`TestCheckAddedDecisionsKept` gains rows: a record that drops an appended bullet a version held yields one `pattern-change-dropped` finding naming it; a record that keeps it, or a version whose bullet carries no suffix, yields none; a version whose section holds a malformed bullet contributes no bullet; and a record whose own section is malformed, with a version holding an appended bullet it lacks, yields no `pattern-change-dropped` finding.

**ImpactSummary:** `AppendDecision`'s only caller is `lyx loom decision add`, unchanged until card 7; `CheckAddedDecisionsKept`'s two callers (`ValidateDarnDiscussion` and the verb's darn check) gain the bullet drop finding.
