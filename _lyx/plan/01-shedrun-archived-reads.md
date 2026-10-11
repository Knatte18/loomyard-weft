# Card 1 — shedrun archived reads

**Create:**
- `plan:internal/shedrun#ArchivedSeedPath` -> `func ArchivedSeedPath(runID string) (string, error)`
- `plan:internal/shedrun#ArchivedStatusPath` -> `func ArchivedStatusPath(runID string) (string, error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`
- `plan:internal/shedrun#TestArchivedPaths` -> `func TestArchivedPaths(t *testing.T)`
- `plan:internal/shedrun#TestDecodeSeed` -> `func TestDecodeSeed(t *testing.T)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decision 9: a run's `seed.json` and `status.json` are read from a tree in git, and `internal/shedrun` stays their sole declarer and decoder (`PATTERN-shed-run-directory`).
`ArchivedSeedPath` and `ArchivedStatusPath` return the slash-separated, tree-root-relative path of the file for a run id, built with `path.Join` over `lyxdirs.LyxDirName`, `shedDirName` and the run id, so `tools/tokencount` spells neither the `shed` segment nor `_lyx`.
Each validates the run id with `ValidateRunID` and returns that error for an empty id, one holding a slash or backslash, `.` or `..`; neither applies `ResolveRunID` or the legacy `self` directory fallback, because there is no `*lyxcwd.Location` and no disk, so `self` joins as the literal segment `self`.
`DecodeSeed` is the decode half of `ReadSeed`, moved out as an exported function over bytes: the strict decode through `decodeSeed` (an unknown key refused, the retired `parent` key accepted and dropped), the default of an absent or empty `Driver` to `DriverGo`, `ValidateDriver` on the result, and the refusal of an empty `Params` key.
`ReadSeed` keeps its run-id validation, its file read and its absent-file result, and calls `DecodeSeed` for the rest, so the two readers cannot drift.
The one observable change is wording: a driver or params-key refusal from `ReadSeed` now reads `shedrun: decode seed for run "<id>": ...` instead of `shedrun: seed for run "<id>": ...`; a grep of the repository finds no assertion on either text.
`doc.go` gains a paragraph on the archived pair and `DecodeSeed`, so the package doc names them.
The new tests are `TestArchivedPaths` in `paths_test.go` and `TestDecodeSeed` in `seed_test.go`; each is one table, so the shedrun row in `test-budget.yaml` rises by two.
`TestArchivedPaths` rows: a plain id gives `_lyx/shed/<id>/seed.json` and `.../status.json` with forward slashes; `self` joins as `self`, not the worktree slug; one invalid id, `a/b`, returns an error from each function, which shows the delegation to `ValidateRunID`, whose own vocabulary `TestRunIDVocabulary` already pins.
`TestDecodeSeed` takes only what no existing test reaches, since `TestSeed_LegacyParentKey`, `TestReadSeed_AbsentDriverDefaultsToGo`, `TestWriteSeed_ReadSeed_RoundTrip` and `TestValidateDriver` already cover the strict decode, the retired key, the driver default, `started_at` and the driver vocabulary through `ReadSeed`: rows for a valid seed handed in as bytes, an empty params key and malformed JSON, the last two each returning an error.
The functions sit where their siblings do: the two path functions in `paths.go`, `DecodeSeed` in `seed.go`, and the file headers of both are amended in the same edits, `seed.go`'s sentence on the sole parser to name `DecodeSeed` and `paths.go`'s list of path categories to add the tree-relative archived pair.
The guard this adds is a validation that refuses; nothing it skips or lets through widens, and the bound is `ValidateRunID`'s own single-segment rule.

**ImpactSummary:** `ReadSeed` has callers in loomcli, battencli and the shed verbs whose behavior is unchanged but for the wording of two refusals no test asserts; the two path functions and `DecodeSeed` are additive.
