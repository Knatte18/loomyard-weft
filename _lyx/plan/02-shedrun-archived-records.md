# Card 2 — shedrun-archived-records

**Create:**
- `plan:internal/shedrun#ArchivedRels` -> `func ArchivedRels(runID string) (seedRel, statusRel string, err error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun/doc.go`
- `internal/shedrun#TestPathConstructors`
- `internal/shedrun#TestRunIDVocabulary`
- `internal/shedrun#TestSeed_LegacyParentKey`
- `internal/shedrun#TestReadSeed_AbsentDriverDefaultsToGo`

**Uses:**
- `internal/shedrun#shedDirName`
- `internal/shedrun#ValidateRunID`
- `internal/shedrun#decodeSeed`

**Intent:** Give a reader of a git tree the two run-record paths and the seed decoder without any disk resolution, per `PATTERN-shed-run-directory`.
`ArchivedRels(runID)` validates `runID` with `ValidateRunID`, refuses on failure, and returns the slash-separated, anchor-relative paths of the run's `seed.json` and `status.json`: `path.Join` of `lyxdirs.LyxDirName`, `shedDirName`, `runID` and the file name.
It applies no `self` alias and no legacy `self` directory fallback, since the tree it names is not on disk, and its doc comment says so and names `SeedRel` and `StatusRel` as the on-disk forms.
It goes in `paths.go` beside `SeedRel`.
`DecodeSeed(data)` is the strict decoder `ReadSeed` applies to the bytes it read: `decodeSeed`, then the absent-driver default to `DriverGo`, `ValidateDriver` and the non-empty `Params` key check, in that order, with the same error texts.
`ReadSeed` keeps its signature and calls `DecodeSeed` after reading the file, so the two cannot drift.
`doc.go` gains one sentence saying that `ArchivedRels` and `DecodeSeed` serve a reader of a run's records in a git tree.
Tests: two rows in `TestPathConstructors` for the seed and status halves of `ArchivedRels("self")`, whose want is the slash-joined path under `_lyx/shed/self/`, pinning that the literal `self` is not aliased to the worktree name;
an assertion in `TestRunIDVocabulary` that `ArchivedRels` refuses exactly the run-ids `ValidateRunID` refuses;
a subtest in `TestSeed_LegacyParentKey` that `DecodeSeed` accepts the legacy `parent` key and refuses another unknown key;
and an assertion in `TestReadSeed_AbsentDriverDefaultsToGo` that `DecodeSeed` over the same bytes defaults the driver to `DriverGo`.
No new top-level test, so the package stays within its budget row.

**ImpactSummary:** Adds two exported shedrun helpers and routes ReadSeed's decode through one of them; every existing shedrun caller keeps its behavior.
