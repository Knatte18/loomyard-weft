# Card 1 — shedrun reads for a tree from git

**Create:**
- `plan:internal/shedrun#RunRecordRels` -> `func RunRecordRels(runID string) (seedRel, statusRel string, err error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`
- `plan:internal/shedrun#TestRunRecordRels` -> `func TestRunRecordRels(t *testing.T)`
- `plan:internal/shedrun#TestDecodeSeed` -> `func TestDecodeSeed(t *testing.T)`

**Edit:**
- `internal/shedrun/doc.go`
- `pattern/PATTERN-shed-run-directory.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** `tools/tokencount` reads a finished run's `seed.json` and `status.json` out of the weft repository's tree, where no `*lyxcwd.Location` exists, so `shedrun` gains the two declarations that read needs, and no other package names the `shed` segment or `seed.json`.
`RunRecordRels` goes in `paths.go` and returns the slash-separated tree-relative paths of the run's `seed.json` and `status.json`, joined from `lyxdirs.LyxDirName`, the `shedDirName` segment and the run id with `path.Join`.
It calls `ValidateRunID` first and returns that error unchanged; it applies no `self` alias and no legacy `self` directory fallback, so `RunRecordRels("self")` names the `self` directory literally.
`DecodeSeed` goes in `seed.go` and returns `decodeSeed(data)`, so a tree read decodes with the same strict decoding as `ReadSeed`: an unknown key is an error, the retired `parent` key is accepted and dropped, and `started_at` is carried.
It applies no driver default and no driver validation, which `ReadSeed` keeps for the on-disk read.

The refusal in `RunRecordRels` is reached by a run id holding a path separator, such as `a/b`, which passes no upstream check because the function has none ahead of `ValidateRunID`; `TestRunRecordRels` has that row.
The error in `DecodeSeed` is reached by a seed holding an unknown key, such as `{"recipe":"loom","bogus":1}`, and by bytes that are not JSON; `TestDecodeSeed` has both rows.
Neither declaration adds a bypass: `RunRecordRels` only builds a string and reads no disk, and a caller that skips it cannot name the segment because no other package declares it.

`TestRunRecordRels` is one table: a plain run id gives `_lyx/shed/<id>/seed.json` and `_lyx/shed/<id>/status.json`, `self` gives the `self` paths unaliased, and `a/b` returns the `ValidateRunID` error with empty paths; the validator's own cases stay in `TestRunIDVocabulary`.
`TestDecodeSeed` is one table: `started_at` is carried, a seed with the retired `parent` key decodes, a seed with another unknown key is refused, and non-JSON is refused.
Both are new top-level tests because no existing `shedrun` test calls a function with these shapes: `TestPathConstructors` takes a `*lyxcwd.Location`, and `TestSeed_LegacyParentKey` drives `ReadSeed` over a file.

Docs in the same commit: `doc.go` gains a paragraph saying a tree read from git uses `RunRecordRels` and `DecodeSeed`, written one sentence per line.
`pattern/PATTERN-shed-run-directory.md` gains a bullet under its list saying a caller that reads a run's records from a git tree takes the paths from `RunRecordRels` and the seed from `DecodeSeed`, and never joins the segment or decodes `Seed` itself.
The `internal/shedrun` row in `test-budget.yaml` goes from 22 to 24, for the two new top-level tests.

**ImpactSummary:** Additive declarations in `internal/shedrun` with no caller yet; the budget row and two doc files change with them, and no existing function changes behavior.
