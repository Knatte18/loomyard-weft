# Discussion check refuses a PATTERN text over the entry line cap

A decision record whose `## PATTERN changes` bullet declares an `adds` or `changes` text too long for a PATTERN.md entry line used to pass Discussion and first fail at Plan-Write, where `pattern-entry-line-cap` refuses the fenced line.
The run then waited on the parent to replace the bullet.
`internal/discussionparser`'s `Validate` now measures the overview line each such bullet prescribes, `` - `PATTERN-<id>` — <text> ``, in runes, and reports one `pattern-change-over-cap` finding per bullet over 360.
The finding names the entry, the line length and the cap, and gives a way forward.

`lyx loom decision add --pattern-text` refuses an over-cap text through the same check: `AppendDecision` runs `Validate` after the write and restores the record on a finding.
`contracts/specs/loom-discussion-spec.md` states the cap in its PATTERN changes section, and the package `doc.go` names the check.

## Design decisions

- The check lives in `Validate`, not in `ParsePatternChanges`.
  `Validate` is the one function the loom discussion gates, the darn gate, `lyx loom validate-discussion` and `AppendDecision` all run, so one site covers every path.
  `ParsePatternChanges` stays a shape check, so `DeclaredPatternChanges`, the plan gate's mismatch check and the dropped-bullet check still read an over-cap bullet as well-formed.
- The measured line is the one Plan-Write must fence, counted in runes with the text trimmed and without the appended-after-Discussion suffix, so it agrees with `pattern-entry-line-cap` on every bullet.
- `internal/discussionparser` declares its own unexported `patternEntryLineCap` (360), because its leaf test admits only the standard library in production code and `internal/pattern` pulls in the stencil packages.
  `TestPatternEntryLineCap_EqualsThePatternCap` pins it to `pattern.MaxEntryLineChars`, so a drift of either value fails the test.
- A bullet appended by `lyx loom decision add` is measured like any other.
  Its way forward tells the parent to re-record the entry with a shorter `--pattern-text`, which replaces the bullet in place.
- A `removes` bullet prescribes no line and is never measured.

## Risks and manual checks

- A record already holding an over-cap appended bullet now fails the discussion check until its parent re-records the entry.
  No live run is known to hold one.
- The tests were not run by hand.
