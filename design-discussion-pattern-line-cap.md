# Discussion check refuses a declared PATTERN text over the overview line cap

`discussion-pattern-line-cap` · Task · bug, loom · running · Darn-Write

## Seen (orch, 2026-10-23)

In `loom-tail-review`, the decision record declared a `changes` text for `PATTERN-parent-lock` of 392 runes as an overview line.
Discussion's check passed it; Plan-Write then could not fence it, since `pattern-entry-line-cap` (`internal/planparser/patternentry.go`) caps a prescribed PATTERN entry line at `pattern.MaxEntryLineChars` (360), while `pattern-changes-mismatch` and `pattern-line-unprescribed` forbid any other text.
The run waited on the parent to replace the record's bullet.

## Idea

The discussion check (`internal/discussionparser`) applies the same cap to each `## PATTERN changes` bullet's `adds` and `changes` text, so Discussion-Write shortens it before its review, and `lyx loom decision add --pattern-text` refuses an over-cap text too.
