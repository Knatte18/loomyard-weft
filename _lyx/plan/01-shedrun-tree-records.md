# Card 1 — shedrun reads a run's records out of a git tree

**Rename:**
- `internal/shedrun#TestReadSeed_AbsentDriverDefaultsToGo` -> `plan:internal/shedrun#TestSeedDefaultsAndValidation`

**Create:**
- `plan:internal/shedrun#TreeSeedRel` -> `func TreeSeedRel(runID string) (string, error)`
- `plan:internal/shedrun#TreeStatusRel` -> `func TreeStatusRel(runID string) (string, error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`
- `plan:internal/shedrun#validateSeed` -> `func validateSeed(seed *Seed) error`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun#TestRunsRootRel`
- `internal/shedrun#TestSeed_LegacyParentKey`
- `internal/shedrun/doc.go`

**Intent:**
Decision 9: `tools/tokencount` reads an archived run's `seed.json` and `status.json` out of a weft tag's tree, so `shedrun` gains the tree-side path pair and the seed decoder, and stays the sole declarer of the `shed` segment and `seed.json` and the sole decoder of `Seed` (`PATTERN-shed-run-directory`).

`TreeSeedRel` and `TreeStatusRel` (in `paths.go`) validate `runID` with `ValidateRunID` and return the slash-separated tree path `path.Join(lyxdirs.LyxDirName, shedDirName, runID, "seed.json")` (and `"status.json"`), the form `gitrepo.Repo.FileAtRevision` takes.
They take no `*lyxcwd.Location` and apply no `self` alias and no legacy `self` fallback: `"self"` joins the literal segment `self`.
A run id that fails `ValidateRunID` (for example `"a/b"` or `""`) returns that error and no path.
Their doc comments say they address a tree read from git, never the disk, which is why they resolve nothing.

`DecodeSeed` (in `seed.go`) is the strict seed decoder over bytes, the same decoding `ReadSeed` applies to the file: `decodeSeed` (unknown keys refused, the retired `parent` key accepted and dropped), then `validateSeed`.
`validateSeed` (in `seed.go`) holds the checks `ReadSeed` runs after decoding today, moved verbatim: an empty `Driver` defaults to `DriverGo`, the driver passes `ValidateDriver`, and no `Params` key is empty.
`ReadSeed` keeps its run-id validation and its file read, then calls `decodeSeed` and `validateSeed` in turn, wrapping each error exactly as it does now (`shedrun: decode seed for run %q: ...`, `shedrun: seed for run %q: ...`), so its observable errors are unchanged.
`WriteSeed` keeps calling `decodeSeed` for its agreement check.

Comments that name the seed's sole reader, or describe every path constructor, are corrected in the same commit: the `seed.go` file comment and the package doc in `doc.go` name `DecodeSeed` beside `ReadSeed`/`WriteSeed` as the seed's parsers, and the `paths.go` file comment says the tree-relative pair joins no location and returns slash-separated paths for a git tree.

Tests, all rows in existing tests:
- `TestRunsRootRel` gains rows: `TreeSeedRel("x")` is exactly `"_lyx/shed/x/seed.json"` built from `lyxdirs.LyxDirName`, `TreeStatusRel("x")` exactly `"_lyx/shed/x/status.json"`, `TreeSeedRel("self")` is `"_lyx/shed/self/seed.json"` (no alias), and `TreeSeedRel("a/b")` and `TreeStatusRel("")` each return a non-nil error and an empty path.
- `TestSeed_LegacyParentKey` gains a subtest: `DecodeSeed` of `{"recipe":"loom","driver":"go","parent":"ab:hub"}` returns `Seed{Recipe: RecipeLoom, Driver: DriverGo}`, and `DecodeSeed` of the same with `"bogus":"x"` returns an error.
- `TestReadSeed_AbsentDriverDefaultsToGo` is renamed `TestSeedDefaultsAndValidation`, since it now pins the post-decode checks of both readers, and gains rows: `DecodeSeed` of `{"recipe":"loom"}` returns `Driver == DriverGo`, of `{"recipe":"loom","driver":"nope"}` returns an error, and of `{"recipe":"loom","params":{"":"x"}}` returns an error.

**ImpactSummary:** Additive to `internal/shedrun`; `ReadSeed` keeps its signature and error wording, and its callers across loom, batten and shed see no change.

**Commit:** 1: shedrun: tree-relative seed and status paths and DecodeSeed
