# Card 1 — shedrun reads a run's records from a tree

**Create:**
- `plan:internal/shedrun#ArchivedSeedRel` -> `func ArchivedSeedRel(runID string) (string, error)`
- `plan:internal/shedrun#ArchivedStatusRel` -> `func ArchivedStatusRel(runID string) (string, error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun#TestRunsRootRel`
- `internal/shedrun#TestSeed_LegacyParentKey`
- `internal/shedrun/doc.go`
- `internal/shedrun/paths.go`

**Intent:**
Decision 9: `tools/tokencount` reads a finished run's `seed.json` and `status.json` out of a git tree, where `SeedRel` and `StatusRel` do not fit, since they take a `*lyxcwd.Location` and resolve the `self` alias and the legacy `self` directory against the disk.
`shedrun` gains the run-id-relative path pair and a seed decoder over bytes, so the tool names neither the `shed` segment nor `seed.json` and never decodes `Seed` itself (`PATTERN-shed-run-directory`, `PATTERN-lyxdirs-single-declarer`).

`ArchivedSeedRel` and `ArchivedStatusRel` live in `paths.go`.
Each validates `runID` with `ValidateRunID`, returning its error unchanged, and returns the slash-separated `path.Join` of `lyxdirs.LyxDirName`, `shedDirName`, `runID` and the file name: `_lyx/shed/<runID>/seed.json` and `_lyx/shed/<runID>/status.json`, the form `gitrepo.Repo.FileAtRevision` takes.
They apply no alias and no legacy `self` fallback: `self` is joined as the literal segment it is.

`DecodeSeed` lives in `seed.go` and holds the decoding `ReadSeed` does after its file read: `decodeSeed`'s strict decode with the retired `parent` key accepted, an absent or empty `Driver` defaulted to `DriverGo`, then `ValidateDriver` and the non-empty `Params` key check.
`ReadSeed` keeps its run-id validation and file read, calls `DecodeSeed`, and wraps its error as `shedrun: seed for run %q: %w`; no test asserts the old wording (grep for `seed for run` and `decode seed for run` in `_test.go` files finds none).

The doc comments that say every constructor is a `filepath.Join` onto a told location's `AnchorPath()` — the package doc in `doc.go` and the file comment of `paths.go` — gain a sentence naming the two archived-record paths as the exception: run-id-relative, slash-separated, for reading a run's records out of a git tree.
The paths carry no anchor prefix: they name the records at the tree's root, as `planparser.PlanDirRel` and `websterengine.DirRel` do for the tool's existing weft reads.
The comments naming `ReadSeed`/`WriteSeed` as the seed's sole parser and writer — the package doc in `doc.go` and the file comment of `seed.go` — name `DecodeSeed` beside `ReadSeed` as a parser.

Tests:
- `TestRunsRootRel` gains assertions: `ArchivedSeedRel("x")` and `ArchivedStatusRel("x")` equal `_lyx/shed/x/seed.json` and `_lyx/shed/x/status.json` exactly, and each refuses `".."` with an error; `TestRunIDVocabulary` already covers the other invalid ids.
- `TestSeed_LegacyParentKey` gains a subtest: `DecodeSeed` of the legacy bytes `{"recipe":"loom","driver":"go","parent":"ab:hub"}` returns the seed `ReadSeed` returns for the same file, and `DecodeSeed` of `{"recipe":"loom"}` has `Driver` `go`; the unknown-key refusal stays covered by the existing subtest through `ReadSeed`, which now calls `DecodeSeed`.

**ImpactSummary:** `ReadSeed` keeps its signature and behavior for its callers across the module; the new functions have no caller until card 3, and the package stays within its test-budget row.

**Commit:** 1: shedrun: run-id-relative seed and status paths and a seed decoder over bytes
