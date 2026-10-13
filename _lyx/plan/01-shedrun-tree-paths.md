# Card 1 — shedrun-tree-paths

**Create:**
- `plan:internal/shedrun#SeedTreeRel` -> `func SeedTreeRel(runID string) (string, error)`
- `plan:internal/shedrun#StatusTreeRel` -> `func StatusTreeRel(runID string) (string, error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun#TestRunsRootRel`
- `internal/shedrun#TestSeed_LegacyParentKey`
- `internal/shedrun#TestReadSeed_AbsentDriverDefaultsToGo`
- `internal/shedrun/doc.go`

**Intent:**
Give `internal/shedrun` the read side a tree read from git needs (Decision 9), so `tools/tokencount` names neither the `shed` segment nor `seed.json` and decodes no `Seed` itself (`PATTERN-shed-run-directory`, `PATTERN-lyxdirs-single-declarer`).

`SeedTreeRel` and `StatusTreeRel`, in `paths.go`, validate `runID` with `ValidateRunID` and return the slash-separated, tree-relative path `path.Join(lyxdirs.LyxDirName, shedDirName, runID, "seed.json")` (and `"status.json"`), the form `gitrepo.Repo.FileAtRevision` takes on every OS.
They apply no alias and no legacy `self` fallback: `self` joins as the literal segment `self`, because no `*lyxcwd.Location` and no disk is involved.
An invalid run id returns the `ValidateRunID` error and an empty path.

`DecodeSeed`, in `seed.go`, is the on-disk reader's decoding over bytes: `decodeSeed`'s strict decode (an unknown key is an error, the retired `parent` key is accepted and dropped), an absent or empty `Driver` defaulted to `DriverGo`, then `ValidateDriver` and the non-empty `Params` key check.
`ReadSeed` keeps its run-id validation and its file read, and calls `DecodeSeed` for the rest, wrapping its error as `shedrun: seed for run %q: %w`; no caller or test outside `seed.go` matches the old `decode seed for run` wording (grepped).

Tests, no new top-level test (the `internal/shedrun` budget row is full):
- `TestRunsRootRel`, which already pins `SeedRel` and `StatusRel` under the run-records root, gains the tree pair: `SeedTreeRel("x")` is `_lyx/shed/x/seed.json` and `StatusTreeRel("self")` is `_lyx/shed/self/status.json`, both spelled with `/` whatever the OS (`filepath.Join` would not be) and the second showing no alias applies, and `SeedTreeRel("a/b")` and `StatusTreeRel("..")` return an error and an empty path.
- `TestSeed_LegacyParentKey` gains `DecodeSeed` assertions on the same bytes its subtests already write: the legacy `parent` seed decodes, and the seed with the extra unknown key returns an error.
- `TestReadSeed_AbsentDriverDefaultsToGo` gains the assertion that `DecodeSeed` of a seed with no `driver` key returns `DriverGo`, and that a seed whose driver is `"bogus"` returns an error.

`doc.go` gains one sentence: the tree-relative path pair and `DecodeSeed` serve a run's records read from a git tree, with no alias resolution.

**ImpactSummary:** additive exports in `internal/shedrun`; `ReadSeed` keeps its signature and behaviour, only its decode-error wording changes, which no caller or test matches.
