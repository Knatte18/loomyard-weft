# Card 1 — shedrun reads a run's records from a git tree

**Create:**
- `plan:internal/shedrun#RunFilesRel` -> `func RunFilesRel(runID string) (seedRel, statusRel string, err error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun/doc.go`
- `internal/shedrun#TestRunsRootRel`
- `internal/shedrun#TestReadSeed_AbsentDriverDefaultsToGo`
- `internal/shedrun#TestSeed_LegacyParentKey`

**Intent:**
Give a reader of a run's records in a git tree, `tools/tokencount` in card 3, the paths and the seed decoder it needs without naming the `shed` segment or `seed.json` itself (Decision 9, `PATTERN-shed-run-directory`, `PATTERN-lyxdirs-single-declarer`).

`RunFilesRel`, in `paths.go`, validates `runID` with `ValidateRunID` and returns the error unchanged when it fails.
Otherwise it returns the slash-separated, anchor-relative paths of the run's `seed.json` and `status.json`: `path.Join(lyxdirs.LyxDirName, shedDirName, runID, <file>)`, the slash form `gitrepo`'s `FileAtRevision` takes.
It applies no `self` alias and no legacy `self` fallback, since a git tree has no disk to probe; it joins the same `shedDirName` declaration the on-disk paths join.

`DecodeSeed`, in `seed.go`, is `ReadSeed`'s decode half over bytes: `decodeSeed`'s strict decode (unknown key refused, the retired `parent` key discarded), an absent or empty `Driver` defaulted to `DriverGo`, then `ValidateDriver` and the non-empty `Params` key check.
`ReadSeed` keeps its run-id validation and its absent-file answer, and replaces its decode, default and validation steps with one `DecodeSeed` call wrapped as `shedrun: decode seed for run %q: %w`.
That rewords `ReadSeed`'s driver and params refusals from `shedrun: seed for run %q: ...` to `shedrun: decode seed for run %q: ...`; a grep for `seed for run` finds no test or caller outside `internal/shedrun/seed.go` asserting the old text.
`WriteSeed` keeps calling `decodeSeed`.

`doc.go` gains one paragraph: `RunFilesRel` takes no `*lyxcwd.Location` because it addresses a run in a git tree, so it applies no alias or legacy fallback, and `DecodeSeed` is the decoder for seed bytes read from anywhere but `SeedFile`.
The sentence saying every constructor joins onto a told location's `AnchorPath()` gains the `RunsRootRel` and `RunFilesRel` exceptions.
`seed.go`'s file comment names `DecodeSeed` among the sole parser's entry points.

Tests, all rows of existing tests since `internal/shedrun` sits at its budget row:
- `TestRunsRootRel` asserts `RunFilesRel("x")` returns `_lyx/shed/x/seed.json` and `_lyx/shed/x/status.json` (slash-separated, spelled through `lyxdirs.LyxDirName`), that `RunFilesRel("self")` returns `_lyx/shed/self/...` unaliased, and that `RunFilesRel("a/b")` returns an error and empty paths, which shows the validation error passes through; `TestRunIDVocabulary` already covers the rest of `ValidateRunID`'s vocabulary.
- `TestReadSeed_AbsentDriverDefaultsToGo` asserts `DecodeSeed` over the same `{"recipe":"loom"}` bytes defaults `Driver` to `DriverGo` with no `StartedAt`, that `DecodeSeed` over `{"recipe":"loom","started_at":"2026-01-02T03:04:05Z"}` keeps that `StartedAt`, and that `DecodeSeed` refuses `{"recipe":"loom","driver":"bogus"}`.
- `TestSeed_LegacyParentKey`'s "another unknown key is still refused" subtest asserts `DecodeSeed` refuses the same bytes, which its doc comment already describes.

**ImpactSummary:** `ReadSeed` keeps its signature and its callers; the two new exports have no caller until card 3, and only `ReadSeed`'s driver and params refusal prefix changes, which nothing outside `seed.go` asserts.

**Commit:** 1: shedrun: tree-relative run file paths and a seed decoder over bytes
