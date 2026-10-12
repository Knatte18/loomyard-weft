# Card 1 — shedrun-tree-files

**Create:**
- `plan:internal/shedrun#TreeFiles` -> `func TreeFiles(runID string) (seed, status string, err error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`
- `plan:internal/shedrun#TestTreeFiles` -> `func TestTreeFiles(t *testing.T)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun#TestSeed_LegacyParentKey`
- `internal/shedrun#TestReadSeed_AbsentDriverDefaultsToGo`
- `internal/shedrun/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** `tools/tokencount` reads a finished run's `seed.json` and `status.json` from a git tree, where `SeedRel` and `StatusRel` do not fit: they take a `*lyxcwd.Location` and resolve the `self` alias and the legacy `self` directory against the disk.
`TreeFiles` in `paths.go` returns the slash-separated, tree-root-relative paths of the two files for a run id, `_lyx/shed/<runID>/seed.json` and `_lyx/shed/<runID>/status.json`, built with `path.Join` from `lyxdirs.LyxDirName` and `shedDirName` so `shedrun` stays the sole declarer of the `shed` segment.
It calls `ValidateRunID` first and returns its error for an id that is not one clean segment, and it applies no alias: `self` names the directory `self`.
`DecodeSeed` in `seed.go` is the strict decode over seed bytes that `ReadSeed` runs on a file's bytes: `decodeSeed`, then an empty `Driver` defaulting to `DriverGo`, `ValidateDriver` on the result and a non-empty `Params` key check.
`ReadSeed` keeps its signature and calls `DecodeSeed` after its read, wrapping any error as `shedrun: decode seed for run %q`; the driver and params refusals move into `DecodeSeed` unchanged, and only their error prefix differs, which no test or caller reads.
The doc comment in `doc.go` gains a paragraph saying `TreeFiles` and `DecodeSeed` serve a reader of a git tree, with no alias and no legacy fallback.
Nothing is skipped or let through that a disk read refused: an invalid run id is refused before any path is built, and a seed with an unknown key or an unknown driver is refused as it is on disk.
The refusal branches are reached by `TestTreeFiles` rows for the ids `""`, `a/b`, `a\b`, `.` and `..`, a row for `my-task` and a row for `self` that stays `self`; by a `TestSeed_LegacyParentKey` subtest decoding a seed with a retired `parent` key through `DecodeSeed` (accepted, the public use `ReadSeed` already reaches only through the file); and by a `TestReadSeed_AbsentDriverDefaultsToGo` subtest decoding `{"recipe":"loom"}` to `DriverGo` and `{"recipe":"loom","driver":"bogus"}` to an error.
`cmd/lyx/testdata/test-budget.yaml` raises the `internal/shedrun` row by one for `TestTreeFiles`.

**ImpactSummary:** `ReadSeed` keeps its signature and its callers outside `shedrun` see the same results; only the prefix of a driver or params refusal changes, and no test or caller matches it.
