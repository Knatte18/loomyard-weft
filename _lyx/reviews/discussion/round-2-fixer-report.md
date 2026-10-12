# Round 2 fixer-report — discussion

The review's line numbers sat about eleven lines below the file's current lines.
Every finding was located by its quoted text.

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  D5's Severity rule now ends at the closing parenthesis, so it is a noun phrase in D4's slot.
  D3's fourth bullet now adds the line "Report defects only, never a passed check. Give each finding's evidence as the `path:line` or glyph read." to every fork's prompt.
  The fork-boilerplate acceptance criterion now gives the exact rendered line in place of "as one well-formed sentence", so a test can pin it.
- **F2 (LOW), fixed as proposed.**
  The `fabric-vocabulary-name` refuses criterion gains the row "a `Type.Method` member under `internal/` whose type segment alone carries a side token".
- **F3 (LOW), fixed as proposed.**
  The fork give-up bullet is removed from Open risks.
  D2 still states the risk and its bound.

## Disputed

None.

## Deferred

None.
