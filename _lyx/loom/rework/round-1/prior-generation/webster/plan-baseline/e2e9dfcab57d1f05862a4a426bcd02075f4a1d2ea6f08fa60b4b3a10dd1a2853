# Card 2 — shed-record-parsers

**Create:**
- `plan:internal/shedrun#ParseSeedTolerant` -> `func ParseSeedTolerant(data []byte) (Seed, error)`
- `plan:internal/shedrun#RunTreeRels` -> `type RunTreeRels struct`
- `plan:internal/shedrun#TreeRunRels` -> `func TreeRunRels(runID string) (RunTreeRels, error)`
- `plan:internal/shedengine#ParseStatusTolerant` -> `func ParseStatusTolerant(data []byte) (Status, error)`

**Edit:**
- `internal/shedrun#TestSeed_LegacyParentKey`
- `internal/shedrun#TestRunsRootRel`
- `internal/shedengine#TestStatus_JSONRoundTrip`

**Intent:**
D5 gives the report its own tolerant parse-from-bytes functions in the packages that own the shed run records, and D4 needs every run record's tree-relative path.

`shedrun.ParseSeedTolerant` (in `seed.go`) decodes a `seed.json` read out of a git tree with plain `json.Unmarshal`, so an unknown key reads; it defaults an empty `Driver` to `DriverGo` like `ParseSeed` and validates nothing else, and returns an error only for bytes that are not a JSON object.
`ParseSeed`, `decodeSeed` and `ReadSeed` stay byte-for-byte unchanged, and the report never calls them (D5's bound).
`shedrun.RunTreeRels` (in `paths.go`) holds the slash-separated, tree-relative paths of one run's records: `Dir`, `Seed`, `Status`, `DriveReports`, `Arbiter` and `Alignment`.
`shedrun.TreeRunRels` returns them for `runID`, built with `path.Join` from `lyxdirs.LyxDirName`, the shed segment, the run id and the package's own `driveReportsDirName`, `arbiterDirName` and `alignmentDirName`, with no `self` alias and no legacy fallback, exactly like `TreeRunFileRels`; an invalid run id is `ValidateRunID`'s error and a zero value.
`shedengine.ParseStatusTolerant` (in `status.go`) decodes a `status.json`'s bytes with `json.Unmarshal`, so an unknown key reads, and returns the decode error otherwise; it checks no `State` value, since the report reads finished and older records the live read gate would refuse.

Tests:
- `TestSeed_LegacyParentKey`'s "another unknown key is still refused" subtest keeps its `ReadSeed` refusal and adds that `ParseSeedTolerant` over the same bytes returns recipe `loom` and driver `go` with no error, which is the acceptance criterion "a seed with an unknown field reads through the report's tolerant parse, and `shedrun.ReadSeed` still rejects it".
  The same subtest adds that `ParseSeedTolerant([]byte("[1]"))` is an error.
- `TestRunsRootRel` gains rows asserting `TreeRunRels("my-slug")` equals the slash forms of `RunDirRel`, `SeedRel`, `StatusRel`, `DriveReportsRel`, `ArbiterRel` and `AlignmentRel` at a synthetic location for the same id, and that `TreeRunRels("a/b")` is an error with a zero value.
- `TestStatus_JSONRoundTrip` adds that the marshalled `want` with an extra top-level `"bogus": 1` key parses through `ParseStatusTolerant` to the same `CurrentProducer`, `State` and `History`, and that `ParseStatusTolerant([]byte("{"))` is an error.

**ImpactSummary:** Three test functions gain assertions; no existing production member changes, so no live reader's acceptance moves.

**Commit:** `2: tolerant shed record parsers and tree paths`
