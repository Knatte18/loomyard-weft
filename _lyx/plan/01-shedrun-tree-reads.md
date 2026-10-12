# Card 1 — shedrun tree reads

**Create:**
- `plan:internal/shedrun#TreeFilesRel` -> `func TreeFilesRel(runID string) (seedRel, statusRel string, err error)`
- `plan:internal/shedrun#DecodeTreeSeed` -> `func DecodeTreeSeed(data []byte) (Seed, error)`

**Edit:**
- `internal/shedrun#TestRunsRootRel`
- `internal/shedrun#TestSeed_LegacyParentKey`
- `internal/shedrun/doc.go`

**Intent:** `PATTERN-shed-run-directory` makes `internal/shedrun` the sole declarer of the `shed` segment and `seed.json`, and the sole decoder of `Seed`, so `tools/tokencount` reaches a run's records in a git tree only through these two additions, in `paths.go` and `seed.go`.
`TreeFilesRel` returns the slash-separated, run-id-relative paths of `seed.json` and `status.json` for a tree read from git, joined from `lyxdirs.LyxDirName`, the same `shedDirName` the on-disk constructors use, the run id and the file name.
It validates the run id with `ValidateRunID` and returns its error unchanged; it takes no `*lyxcwd.Location`, so it applies no alias and no legacy `self` fallback, and the run id `self` names the `self` directory like any other.
`DecodeTreeSeed` decodes seed bytes with `decodeSeed`, the strict decoder `ReadSeed` uses, so an unknown key is an error and the retired `parent` key is accepted and dropped; it applies neither `ReadSeed`'s driver default nor its driver validation, because a tree reader needs the seed's `StartedAt` only.
What this can let through: a caller reads a record from any revision's tree; the bound is that the run id is validated and the paths carry no `..` or separator.
The refusal branch is `ValidateRunID`'s: the input `a/b` reaches it, since nothing upstream of `TreeFilesRel` checks the id, and a subtest of `TestRunsRootRel` asserts the error and the empty paths.
The package doc in `doc.go` says every constructor joins onto a told location's anchor and describes the `self` alias, so it gains a sentence that `TreeFilesRel` and `DecodeTreeSeed` serve trees read from git, with no alias and no legacy fallback.
Tests extend existing top-level tests, so the `internal/shedrun` budget row stays as it is.
`TestRunsRootRel` gains subtests: `TreeFilesRel("x")` returns `_lyx/shed/x/seed.json` and `_lyx/shed/x/status.json` strictly under `RunsRootRel()`; `TreeFilesRel("self")` returns the `self` directory unaliased; `TreeFilesRel("a/b")` and `TreeFilesRel("")` return an error.
`TestSeed_LegacyParentKey` gains subtests: `DecodeTreeSeed` of a seed holding `started_at` and the `parent` key returns the `StartedAt` and drops the key; one holding another unknown key returns an error.

**ImpactSummary:** Two test functions gain subtests and nothing else changes in `internal/shedrun`; the new functions have no caller until card 3, and no existing caller is touched.
