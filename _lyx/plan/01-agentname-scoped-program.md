# Card 1 — agentname-scoped-program

**Edit:**
- `internal/agentname#ParseProgram`
- `internal/agentname#TestFormatParseRoundTrip`
- `internal/agentname#TestValidationRejections`

**Create:**
- `internal/agentname#FormatScopedProgram`

**Intent:**
Add the beam-scoped program sender `<shortname>:<slug>:<NAME>`, which dobby signs with as `ly:<slug>:DOBBY` for a beam (D1); the prime's dobby keeps the two-segment `ly:DOBBY` that `FormatProgram` forms.
`FormatScopedProgram` validates the shortname, the slug with the slug grammar `Format` already applies, and the upper-case name with `validateProgramName`, and joins the three; an empty slug returns `FormatProgram`'s two-segment form.
`ParseProgram` accepts two segments as today and three segments whose middle is a valid slug and whose last is an upper-case program name, and keeps returning `(shortname, name)`, so `IsProgram`, creel's `checkProgramClaim`, `isExternalName` and `isProgramMailbox` read a scoped sender as a program with no change of their own; a three-segment name whose last segment is lower case stays a strand name for `Parse`.
The package doc's "Where names are formed" paragraph names the scoped form.
Tests: `TestFormatParseRoundTrip` gains rows for a scoped program round trip and for the empty-slug form; `TestValidationRejections` gains rows for a bad slug segment, a lower-case last segment and four segments.

**ImpactSummary:** One new former and a wider program parse; every ParseProgram caller reads the same (shortname, name) pair, so no caller changes.
