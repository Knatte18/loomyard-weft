# Card 1 — shedrun-tree-records

**Create:**
- `plan:internal/shedrun#RunRecordRels` -> `func RunRecordRels(runID string) (seedRel, statusRel string, err error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`
- `plan:internal/shedrun#TestRunRecordRels` -> `func TestRunRecordRels(t *testing.T)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun#TestReadSeed_AbsentDriverDefaultsToGo`
- `internal/shedrun#TestSeed_LegacyParentKey`
- `internal/shedrun/doc.go`
- `internal/shedrun/paths.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedrun#ValidateRunID`
- `internal/shedrun#decodeSeed`

**Intent:**
Give `internal/shedrun` the two read-side pieces a run's records need when they come from a git tree rather than the disk (Decision 9), so `tools/tokencount` names no `shed` segment, no `seed.json` and no `_lyx` literal and decodes no `Seed` itself.

`RunRecordRels` goes in `paths.go`.
It validates `runID` with `ValidateRunID` and returns the error unchanged when it fails.
It applies no alias and no legacy `self` fallback: it never calls `ResolveRunID` or `runSegment`, so `"self"` yields `_lyx/shed/self/...` and any other id is joined as given.
Both paths are slash-separated, joined with `path.Join` from `lyxdirs.LyxDirName`, `shedDirName`, the run id and `seed.json` or `status.json`, because `gitrepo`'s `FileAtRevision` takes slash-separated repo-relative paths on every OS.
Its doc comment says it is the git-tree counterpart of `SeedRel` and `StatusRel`, for a tree read at a revision, where no `*lyxcwd.Location` and no disk exist to resolve an alias against.
Update `paths.go`'s file header comment, which says every constructor joins onto a told Location's `AnchorPath()`, to name this one exception.

`DecodeSeed` goes in `seed.go`.
It runs the strict decoder `decodeSeed` (an unknown key is an error, the retired `parent` key is accepted and dropped), defaults an empty `Driver` to `DriverGo`, then validates the driver with `ValidateDriver` and refuses an empty `Params` key, the same checks `ReadSeed` runs today, with errors that name no run id.
`ReadSeed` keeps its signature and its absent-file result, and replaces its inline decode, default and validation with one `DecodeSeed` call whose error it wraps as `shedrun: decode seed for run %q: %w`.
No test or caller matches the old wrapping texts (`grep -rn "seed for run" --include=*.go` hits only `seed.go`).
`WriteSeed` keeps calling `decodeSeed`.
Update `seed.go`'s file header to list `DecodeSeed` beside `ReadSeed`/`WriteSeed`/`List` as the sole parser of the file.
Update the package doc in `internal/shedrun/doc.go`: its last paragraph says every constructor is a plain `filepath.Join` onto a told Location's `AnchorPath()`; name `RunRecordRels` as the run-id-relative pair for a tree read from git, with no alias resolution, and `DecodeSeed` as the decoder over seed bytes such a reader uses.

Tests:
- New `TestRunRecordRels` in `paths_test.go`, table-driven: a slug yields `_lyx/shed/<slug>/seed.json` and `_lyx/shed/<slug>/status.json` exactly, slash-separated; `"self"` yields the `self` segment, not `worktreeName`; `""`, `"a/b"`, `"a\\b"` and `".."` each return the `ValidateRunID` error and empty paths.
- `TestReadSeed_AbsentDriverDefaultsToGo` gains an assertion that `DecodeSeed` over the same bytes, `{"recipe":"loom"}`, returns `DriverGo` and an empty `StartedAt`.
- `TestSeed_LegacyParentKey` gains, in its "another unknown key is still refused" subtest, an assertion that `DecodeSeed` over the same bytes returns an error, and in its first subtest that `DecodeSeed` over the legacy bytes returns the recipe and driver `ReadSeed` returns.
- Raise `internal/shedrun` in `cmd/lyx/testdata/test-budget.yaml` from 22 to 23 for `TestRunRecordRels`.

**ImpactSummary:** `ReadSeed`'s callers see the same results and error kinds with one wrapping text reworded that no test asserts; the two new exports have no caller until card 4.
