# Card 16 — export-plan-format

**Rename:**
- `internal/planparser#recognizedFormat` -> `internal/planparser#RecognizedFormat`

**Intent:** Export `planparser`'s plan-format version constant so `plankit` (card 17) renders through it and a format bump fails at one site (D10).
Only the name changes: the value, `checkFormatRecognized` and the finding text are untouched, and the doc comment's opening is updated to the new name.
