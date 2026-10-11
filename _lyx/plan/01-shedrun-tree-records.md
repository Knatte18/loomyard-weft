# Card 1 — shedrun reads a run's records from a git tree

**Create:**
- `plan:internal/shedrun#TreeRunFileRels` -> `func TreeRunFileRels(runID string) (seedRel, statusRel string, err error)`
- `plan:internal/shedrun#ParseSeed` -> `func ParseSeed(data []byte) (Seed, error)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun/doc.go`
- `internal/shedrun/paths_test.go`
- `internal/shedrun/seed_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedrun#ValidateRunID`
- `internal/shedrun#shedDirName`
- `internal/shedrun#decodeSeed`

**Intent:**
Give a reader of an archived run's records, held in a git tree rather than on disk, the run's `seed.json` and `status.json` paths and a seed decoder, so `tools/tokencount` names neither the `shed` segment nor `seed.json` and never decodes `Seed` itself (decision record, Decision 9; `PATTERN-shed-run-directory`).

`TreeRunFileRels` validates `runID` with `ValidateRunID` and returns the two files' tree-relative paths, slash-separated as `gitrepo`'s tree reads take them: `lyxdirs.LyxDirName`, `shedDirName`, `runID`, then `seed.json` or `status.json`, joined with `path.Join`.
It applies no alias and no legacy `self` fallback, since a tree read from git has no disk to resolve against; a `runID` that fails validation returns that error and empty paths.

`ParseSeed` decodes seed bytes as `ReadSeed` does: strictly through `decodeSeed`, so an unknown key is an error and the retired `parent` key is accepted and discarded; it defaults an absent or empty `Driver` to `DriverGo` and validates the driver and every `Params` key being non-empty.
`ReadSeed` keeps its run-id validation and file read and calls `ParseSeed` for the rest, wrapping its error as `shedrun: seed for run %q: %w`.
That rewords `ReadSeed`'s decode error from `shedrun: decode seed for run %q`; a grep of the repository for `decode seed for run` and `params key must not be empty` finds no test asserting either text.

The package doc in `doc.go` gains one sentence each for `TreeRunFileRels` (tree-relative, no alias, no legacy fallback) and `ParseSeed` (the decoder over bytes a caller read elsewhere, the same strictness as `ReadSeed`).

Tests:
- `TestRunsRootRel` in `paths_test.go` gains assertions that `TreeRunFileRels("x")` returns `_lyx/shed/x/seed.json` and `_lyx/shed/x/status.json` with forward slashes on every platform, that `TreeRunFileRels("self")` returns the `self` directory's paths with no alias, and that `TreeRunFileRels("a/b")` returns an error and empty paths.
- a new table test `TestParseSeed` in `seed_test.go`: a seed carrying `started_at` decodes with it; a seed carrying the retired `parent` key decodes and drops it; an absent driver decodes as `DriverGo`; an unknown key, an unknown driver and an empty params key each return an error.

Raise `internal/shedrun`'s row in `cmd/lyx/testdata/test-budget.yaml` by the new top-level test.

**ImpactSummary:** One additive path constructor and one additive decoder in `shedrun`; `ReadSeed`'s behavior is unchanged apart from its decode error's wording, which no caller or test matches on.
