# Card 1 — agentname-program-senders

**Create:**
- `internal/agentname#External`
- `internal/agentname#FormatProgram`
- `internal/agentname#ParseProgram`
- `internal/agentname#IsProgram`

**Edit:**
- `internal/agentname#TestFormatParseRoundTrip`
- `internal/agentname#TestValidationRejections`
- `pattern/PATTERN-agent-name.md`

**Intent:** Teach `internal/agentname` the non-strand sender names of D4, so creel forms, parses and tells apart every sender through this one package.
A program sender is `<shortname>:<NAME>`, NAME matching `[A-Z][A-Z0-9_]*` (upper-case marks a program, never a strand role, which stays lower-case).
`External` is the reserved name `EXTERNAL`; `FormatProgram(shortname, External)` gives `<shortname>:EXTERNAL`.
`FormatProgram` validates the shortname with `ValidateShortname` and the name with the upper-case grammar, refusing anything else with an error naming the grammar.
`ParseProgram` accepts exactly two segments whose second is upper-case and returns them, refusing a lower-case second segment, a three-segment name and an invalid shortname.
`IsProgram` reports whether a full name parses as a program sender, `EXTERNAL` included; callers that must tell EXTERNAL apart compare the name to `External`.
`Parse` is unchanged, so a program name never parses as a strand name and a strand name never as a program name.
The package stays standard-library only.
`TestFormatParseRoundTrip` gains rows for `ly:BATTEN` and `ly:EXTERNAL` round-tripping through `FormatProgram`/`ParseProgram` with `IsProgram` true, and a strand name answering `IsProgram` false; `TestValidationRejections` gains `ParseProgram` rows refusing `ly:batten` (lower-case name), `ly:x:BATTEN` (three segments) and `LY:BATTEN` (invalid shortname), `FormatProgram` rows refusing `("ly", "batten")`, `("ly", "BAT-TEN")` and `("LY", "BATTEN")`, and a `Parse` row refusing `ly:BATTEN`.
`pattern/PATTERN-agent-name.md` gains a bullet beside its grammar line: a program sender is `<shortname>:<NAME>` with an upper-case name, formed and parsed by `FormatProgram`/`ParseProgram`, and `EXTERNAL` is the reserved name of a sender creel cannot place; strand names are still formed once by reed.

**ImpactSummary:** Additive to agentname: four new exported names and new rows in its two existing tests; no existing caller changes behavior.
