# Card 3 — seed-started-at

**Edit:**
- `internal/shedrun#Seed`
- `internal/shedrun#WriteSeed`
- `internal/shedrun/seed_test.go`
- `internal/shedrun/doc.go`
- `internal/hubgeom#runStartTime`
- `internal/hubgeom/spawnorder_test.go`
- `internal/hubgeom/doc.go`

**Intent:** The seed carries its start time and the spawn order reads it (D10).
`Seed` gains `StartedAt string` with tag `json:"started_at,omitempty"`, RFC 3339 UTC.
`WriteSeed` stamps it with the current time before encoding when the given seed has none and keeps a given one; agreement with an existing seed still compares recipe, driver and params only, so a re-written agreeing seed keeps its original stamp.
`seed_test.go` asserts: a seed written without a stamp reads back with a parseable, non-zero `StartedAt`; a given stamp is kept verbatim; a second agreeing write keeps the first stamp; and a seed file written before the field decodes, as a row of the existing round-trip and legacy tests.
`hubgeom.runStartTime` reads the seed through `shedrun.ReadSeed` and parses `StartedAt`; a missing seed, a seed without the field or one that does not parse is undated, and the file's modification time is never read.
`spawnorder_test.go`'s `TestRunStartTime` writes seeds through `WriteSeed` with and without a stamp and asserts the dated and undated answers, and that a rewritten seed with the same stamp keeps its time whatever its mtime.
`shedrun/doc.go` names the stamp and `hubgeom/doc.go` replaces the modification-time sentence with the stamp rule and its bound: runs seeded before this lands are undated once.

**ImpactSummary:** One new optional seed field written by the sole seed writer; `hubgeom` is its only reader, and no other seed reader or writer changes.
