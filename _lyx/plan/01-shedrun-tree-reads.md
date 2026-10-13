# Card 1 — shedrun paths and seed decoder for a tree read from git

**Create:**
- `plan:internal/shedrun#RunRecordRels` -> `func RunRecordRels(runID string) (seedRel, statusRel string, err error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun#TestRunsRootRel`
- `internal/shedrun#TestSeed_LegacyParentKey`
- `internal/shedrun/doc.go`
- `internal/shedrun/paths.go`

**Intent:**
Decision 9: give a caller reading a run's records out of a git tree the run-id-relative paths of `seed.json` and `status.json` and a seed decoder, so `tools/tokencount` names neither the `shed` segment nor `seed.json` and decodes no `Seed` itself (`PATTERN-shed-run-directory`, `PATTERN-lyxdirs-single-declarer`).

`RunRecordRels` validates `runID` with `ValidateRunID` and returns its error unchanged on a refusal.
It applies no `self` alias and no legacy `self` fallback, and takes no `*lyxcwd.Location`.
It returns slash-separated paths, `path.Join(lyxdirs.LyxDirName, shedDirName, runID, "seed.json")` and the same with `"status.json"`, because `gitrepo`'s `FileAtRevision` takes slash-separated repo-relative paths on every OS.
The `shed` segment comes from the same `shedDirName` constant the on-disk constructors join.

`DecodeSeed` is the strict decoding `ReadSeed` applies after reading the file: `decodeSeed` (unknown key refused, the retired `parent` key accepted and dropped), an absent or empty `Driver` defaulted to `DriverGo`, `ValidateDriver`, and the non-empty `Params` key check.
`ReadSeed` keeps its run-id validation and file read and calls `DecodeSeed` for the rest, wrapping its errors with the run id as it does now, so its behavior and messages stay as they are.

Tests, as rows of existing tests (the package's row in `cmd/lyx/testdata/test-budget.yaml` is full, so no top-level test is added):
- `TestRunsRootRel` gains rows: `RunRecordRels("x")` returns exactly `_lyx/shed/x/seed.json` and `_lyx/shed/x/status.json` in slash form, both under `filepath.ToSlash(RunsRootRel())`; `RunRecordRels("self")` returns `_lyx/shed/self/...`, no alias; `""`, `"a/b"` and `".."` each return a non-nil error and empty paths.
- `TestSeed_LegacyParentKey` gains one `DecodeSeed` row over raw bytes: a driver `"bogus"` is refused.
  The legacy `parent` key, another unknown key, an absent driver and a kept `started_at` are already reached through `ReadSeed`, which now delegates to `DecodeSeed`, by `TestSeed_LegacyParentKey`'s subtests, `TestReadSeed_AbsentDriverDefaultsToGo` and `TestWriteSeed_ReadSeed_RoundTrip`, so no row repeats them.

`RunRecordRels` goes in `paths.go` and `DecodeSeed` in `seed.go`.
Three comments that the new functions make wrong are corrected:
- the last paragraph of the package doc in `doc.go`, which says every constructor is a `filepath.Join` onto a told `*lyxcwd.Location`'s `AnchorPath()`, gains the exception: `RunRecordRels` is the git-tree read side, slash-joined, run-id-relative, with no Location and no alias resolution;
- the `paths.go` file comment, which says the same of every constructor, names the same exception;
- the file comment of `seed.go`, the file `ReadSeed` lives in (listed by its member glyph, since `redundant-file-target` refuses the file beside it), which names `ReadSeed`/`WriteSeed`/`List` as the sole parser and writer of the file, adds `DecodeSeed` as the decoder for seed bytes read elsewhere, such as from a git tree.

**ImpactSummary:** `ReadSeed` keeps its signature and errors, its callers are untouched; two new exported functions add a read-only surface with one consumer, `tools/tokencount`.

**Commit:** 1: shedrun: run-record paths and seed decoder for a tree read from git
