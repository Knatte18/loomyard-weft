# Support log — discussion check refuses an over-cap PATTERN text

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- **The length check inside `ParsePatternChanges`, as a `pattern-change-malformed` finding.**
  Rejected: `DeclaredPatternChanges` and the plan gate's mismatch check refuse on any parse finding, so an over-cap bullet would turn later steps into hard errors instead of one discussion finding.
- **Importing `internal/pattern` into `internal/discussionparser` for `MaxEntryLineChars`.**
  Rejected: the package's leaf test admits no non-stdlib import, and `internal/pattern` pulls the stencil packages in.
- **Checking the cap in `internal/loomcli` and `internal/loomshed` separately.**
  Rejected: two sites to keep in step where `Validate` already runs at every one.
- **Measuring the text alone, not the overview line.**
  Rejected: the plan gate measures the whole fenced line, so a text-only cap would let a bullet within it still fail at Plan-Write.

## Review rounds

_No rounds yet._

## Question ledger

- Should an appended bullet be measured? Self-pick: yes, since `lyx loom decision add` must refuse an over-cap text and a re-record clears the finding.
- Runes or bytes? Self-pick: runes, matching `pattern.Check` and `pattern-entry-line-cap`.
