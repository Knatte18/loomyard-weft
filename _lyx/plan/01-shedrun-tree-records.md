# Card 1 — shedrun-tree-records

**Create:**
- `plan:internal/shedrun#TreeSeedRel` -> `func TreeSeedRel(runID string) (string, error)`
- `plan:internal/shedrun#TreeStatusRel` -> `func TreeStatusRel(runID string) (string, error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`
- `plan:internal/shedrun#TestTreeRelPaths` -> `func TestTreeRelPaths(t *testing.T)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun#TestReadSeed_AbsentDriverDefaultsToGo`
- `internal/shedrun#TestSeed_LegacyParentKey`
- `internal/shedrun/doc.go`

**Uses:**
- `internal/shedrun#ValidateRunID`
- `internal/shedrun#decodeSeed`
- `internal/shedrun#ValidateDriver`

**Intent:**
Give `internal/shedrun` the read side a tree read from git needs (decision record, Decision 9), so `tools/tokencount` can read an archived run's records without naming the `shed` segment or `seed.json` and without decoding `Seed` itself (`PATTERN-shed-run-directory`, `PATTERN-lyxdirs-single-declarer`).

`TreeSeedRel` and `TreeStatusRel`, in `paths.go`, return the slash-separated path of a run's `seed.json` and `status.json` relative to a records tree's root: `path.Join(lyxdirs.LyxDirName, shedDirName, runID, "seed.json")` and the same with `"status.json"`.
Slash-separated because `gitrepo`'s `FileAtRevision` takes slash-separated repo-relative paths.
Each validates `runID` with `ValidateRunID` first and returns its error unchanged.
Neither takes a `*lyxcwd.Location`, so neither applies the `self` alias or the legacy `self` directory fallback: the run id is joined as given.

`DecodeSeed`, in `seed.go`, is the on-disk reader's decoding over bytes: `decodeSeed`'s strict decode (an unknown key is an error, the retired `parent` key is accepted and dropped), then an absent or empty `Driver` defaults to `DriverGo`, then `ValidateDriver` and the non-empty `Params` key check.
`ReadSeed` keeps its run-id validation and its absent-file `(Seed{}, false, nil)`, and delegates everything after the file read to `DecodeSeed`, wrapping its error as `shedrun: decode seed for run %q: %w`; no test asserts the old wording of the driver and params errors (grepped).
`WriteSeed` keeps calling `decodeSeed` for its agreement check.

Tests, at the unit tier:
`TestTreeRelPaths`, new in `paths_test.go`, pins both paths for a plain run id (`_lyx/shed/<id>/seed.json`, `_lyx/shed/<id>/status.json`), pins that `self` joins as `self`, and pins the refusal of an empty id, `..` and an id holding a slash, each returning an error and an empty path.
`TestReadSeed_AbsentDriverDefaultsToGo` gains the assertion that `DecodeSeed` of the same bytes returns `DriverGo` and an empty `StartedAt`, and a row decoding a seed with `started_at` set returns it unchanged.
`TestSeed_LegacyParentKey` gains `DecodeSeed` assertions beside its `ReadSeed` ones: the `parent` key is dropped and another unknown key is a decode error.
The package's test count stays within its `cmd/lyx/testdata/test-budget.yaml` row.

`doc.go` gains a paragraph naming the tree-relative pair and `DecodeSeed` as the read side for records read from git, with no alias and no legacy fallback.

**ImpactSummary:** Adds two location-free path functions and an exported seed decoder to `internal/shedrun`; `ReadSeed`'s behavior is unchanged for every caller, and only its driver and params error wrapping is reworded.

**Commit:** 1: shedrun: tree-relative seed and status paths, DecodeSeed
