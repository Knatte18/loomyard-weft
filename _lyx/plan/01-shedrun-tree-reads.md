# Card 1 — shedrun tree paths and seed decoder

**Create:**
- `plan:internal/shedrun#SeedTreeRel` -> `func SeedTreeRel(runID string) (string, error)`
- `plan:internal/shedrun#StatusTreeRel` -> `func StatusTreeRel(runID string) (string, error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun/doc.go`
- `internal/shedrun#TestRunsRootRel`
- `internal/shedrun#TestReadWriteSeed_InvalidResolvedIDRefused`
- `internal/shedrun#TestValidateDriver`

**Uses:**
- `internal/shedrun#ValidateRunID`
- `internal/shedrun#ValidateDriver`
- `internal/shedrun#decodeSeed`
- `internal/shedrun#RunsRootRel`
- `internal/shedrun#SeedRel`
- `internal/shedrun#StatusRel`

**Intent:**
Decision 9: `tools/tokencount` reads an archived run's `seed.json` and `status.json` out of a git tree, where no `*lyxcwd.Location` exists, so `shedrun` gains a run-id-relative path pair and a decoder over seed bytes, keeping it the sole declarer of the `shed` segment and `seed.json` and the sole decoder of `Seed` (`PATTERN-shed-run-directory`).

`SeedTreeRel` and `StatusTreeRel` live in `paths.go`.
Each validates `runID` with `ValidateRunID` and returns the slash-separated path `path.Join(lyxdirs.LyxDirName, shedDirName, runID, "<file>")`, the tree-path form `gitrepo.Repo.FileAtRevision` takes.
They apply no `self` alias and no legacy `self` fallback: `SeedTreeRel("self")` is `_lyx/shed/self/seed.json`.
A run id that fails validation returns an error that names the file (`seed.json` or `status.json`) and wraps the `ValidateRunID` error, which names the run id, so a caller's error names the file without spelling the filename itself.
`paths.go`'s file comment gains a clause for the two tree paths.

`DecodeSeed` lives in `seed.go` and is the same strict decoding `ReadSeed` applies: `decodeSeed` (unknown keys refused, the retired `parent` key accepted and dropped), an absent or empty `Driver` defaulted to `DriverGo`, then `ValidateDriver` and the non-empty `Params` key check.
`ReadSeed` keeps its run-id validation and file read and calls `DecodeSeed`, wrapping its error as `shedrun: seed for run %q: %w`; no test asserts the old `decode seed for run` wording (grep `decode seed for run` and `seed for run` over `*_test.go` finds none).
`seed.go`'s file comment names `DecodeSeed` beside `ReadSeed`/`WriteSeed`/`List`.

`doc.go`'s last paragraph says every constructor is a `filepath.Join` onto a told `*lyxcwd.Location`; it gains the exception: the two tree paths take no location and join only the run id, for a tree read from git rather than the disk.

Tests, as rows of existing tests:
- `TestRunsRootRel`: `SeedTreeRel("x")` and `StatusTreeRel("x")` equal `filepath.ToSlash` of `SeedRel(l, "x")` and `StatusRel(l, "x")`, and `SeedTreeRel("self")` equals `path.Join(filepath.ToSlash(RunsRootRel()), "self", "seed.json")`, where `SeedRel(l, "self")` resolves to the worktree name.
- `TestReadWriteSeed_InvalidResolvedIDRefused`: `SeedTreeRel("a/b")`, `StatusTreeRel("..")` and `StatusTreeRel("")` each return an error whose text holds the file name (`seed.json` or `status.json`).
- `TestValidateDriver`: each row also decodes `{"recipe":"loom","driver":"<driver>"}` with `DecodeSeed`, which fails exactly when the row's `wantErr` is set.

Driver defaulting, the legacy `parent` key and unknown-key refusal reach `DecodeSeed` through `ReadSeed`, whose existing tests (`TestReadSeed_AbsentDriverDefaultsToGo`, `TestSeed_LegacyParentKey`) keep covering them.

**ImpactSummary:** Additive within `internal/shedrun`: `ReadSeed`'s error wrapping changes for its existing callers but none matches on the text; the new functions have no callers until card 3.

**Commit:** 1: shedrun gains tree-relative seed and status paths and DecodeSeed
