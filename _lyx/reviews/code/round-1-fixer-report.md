# Fixer report — round 1, code review

## Changes

- **F1 (MEDIUM), fixed as proposed.** `internal/dotnetimpact/lex.go`: `stringBody` takes the raw-string branch only when `!verbatim`, so a verbatim string opening with an escaped quote runs the verbatim loop.
  Row `VerbatimOpeningWithEscapedQuote_DoesNotSwallowCode` added to `TestParseSource`.
- **F2 (LOW), fixed as proposed.** `internal/dotnetimpact/source.go`: `startsTypeDecl` accepts `record` only when its name is followed by `(`, `<`, `:`, `{` or `;`.
  Row `RecordAsIdentifier_DeclaresNoType` added to `TestParseSource`.
  The self-check then showed `record is { X: 1 }` still passed that check, so a second commit also rejects the names `in`, `is`, `as` and `when`, the review's alternative; the row gained that case.
- **F3 (LOW), fixed as proposed.** `internal/gatecli/doc.go`: the carry sentence names the `dotnet` scoper beside `none` and says a narrowed pass records nothing.

`lyx gate test` over `./internal/dotnetimpact` and `./internal/gatecli` passes.

## Self-check

- fix-code-correctness, LOW: `record is { X: 1 }` still declares a bogus type `is`. Fixed (own change was the review's alternative; second F2 commit).
- fix-proposal, LOW: the same case, with `record is (...)` and `record when (...)`. Merged with the one above and fixed together.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
