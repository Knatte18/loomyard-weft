# Fixer report — plan, round 2

## Changes

- **F1** (LOW, scope), fixed as proposed, in `_lyx/plan/02-claude-transcript-reader.md`:
  - The `MessageKey` rule defines `<n>` in `line:<transcriptKey>:<n>` as the record's 1-based line number in its transcript file.
  - `TestReadSession` gains a row: a main transcript and a fork transcript that each hold an id-less, uuid-less assistant usage line at the same line number yield `MessageKey` values `line:<main ID>:<n>` and `line:<main ID>/<fork ID>:<n>`, which differ.
  - The `TestSessionUsage_SumsParentAndForks` bullet drops the `line:...` keying clauses from both id-less rows.
    It now states that those rows pin only that each id-less line counts, because `SessionUsage` returns sums and parses each transcript with its own per-message map (`transcriptTokens`'s `byID` in `usage.go`).
    The fork-side row stays, but only as the fork-side counting case, and no longer claims a matching parent line number.

## Disputed

None.

## Deferred

None.
