# Card 1 — shedrun reads a run's records out of a git tree

**Create:**
- `plan:internal/shedrun#TreeRecordRels` -> `func TreeRecordRels(runID string) (seedRel, statusRel string, err error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`
- `plan:internal/shedrun#TestDecodeSeed` -> `func TestDecodeSeed(t *testing.T)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun#TestRunsRootRel`
- `internal/shedrun/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decision 9: a tree read from git needs the run's `seed.json` and `status.json` paths and a seed decoder that touch no disk, so `tools/tokencount` names neither the `shed` segment nor `seed.json` and never decodes `Seed` itself (`PATTERN-shed-run-directory`).

`TreeRecordRels`, in `internal/shedrun/paths.go`, validates `runID` with `ValidateRunID` and returns `path.Join(lyxdirs.LyxDirName, shedDirName, runID, "seed.json")` and the same join ending in `"status.json"`.
The joins are slash-separated, the form `gitrepo`'s `FileAtRevision` reads, and anchor-relative like `SeedRel` and `StatusRel`.
It applies no alias and no legacy `self` fallback: it never calls `ResolveRunID` or `runSegment`, so `"self"` joins a `self` segment and any other id joins itself.
An id `ValidateRunID` refuses returns that error and two empty paths.
Its doc comment says it serves a tree read from git, where the on-disk alias and fallback do not apply.

`DecodeSeed`, in `internal/shedrun/seed.go`, holds what `ReadSeed` does to the bytes today: the strict `decodeSeed` (an unknown key is an error, the retired `parent` key is discarded), an absent or empty `Driver` defaulted to `DriverGo`, `ValidateDriver` on it, and the empty-`Params`-key refusal.
`ReadSeed` keeps its run-id validation and file read and calls `DecodeSeed` on the bytes, wrapping its error as `shedrun: seed for run %q: %w`;
no test asserts the old decode or validation wording (a repository grep for `decode seed for run` and `seed for run %q` finds only `internal/shedrun/seed.go`).
`decodeSeed` stays for `WriteSeed`'s existing-seed comparison.

`internal/shedrun/doc.go` gains one sentence: `TreeRecordRels` and `DecodeSeed` read a run's records from a git tree, with the same strict decoding as `ReadSeed` and with no alias or legacy fallback.
Its last paragraph, which says every constructor is a plain `filepath.Join` onto a told location's `AnchorPath()`, and the file comment of `internal/shedrun/paths.go`, which says the same, each name `TreeRecordRels` as the one exception: a slash-separated `path.Join` that takes no location.

Tests:
- `TestRunsRootRel` gains assertions on `TreeRecordRels`: for `"x"` it returns exactly `_lyx/shed/x/seed.json` and `_lyx/shed/x/status.json`, both forward-slashed and under `RunsRootRel()` in slash form; for `"self"` it returns `_lyx/shed/self/seed.json` (no alias to a worktree name); for `"a/b"` it returns a non-nil error and empty paths, which shows `ValidateRunID`'s refusal propagates; `TestRunIDVocabulary` already pins which ids that refuses.
- `TestDecodeSeed`, new in `internal/shedrun/seed_test.go`, is a table over byte inputs, each row asserting the exact `Seed` or a non-nil error.
  It holds only what no `ReadSeed` test reaches: a seed with `recipe`, `driver`, `params` and `started_at` decodes with every field kept; an unknown driver, an empty `params` key and bytes that are not JSON are each refused.
  The absent driver, the retired `parent` key, the unknown key and the kept `started_at` stay covered through `ReadSeed` by `TestReadSeed_AbsentDriverDefaultsToGo`, `TestSeed_LegacyParentKey` and `TestWriteSeed_ReadSeed_RoundTrip`.
  It raises the `internal/shedrun` row of `cmd/lyx/testdata/test-budget.yaml` from 22 to 23.

**ImpactSummary:** two new exported functions in `internal/shedrun`, used by no other package until card 4; `ReadSeed` keeps its behavior and its callers, only its error wrapping changes, and no test asserts that wording.

**Commit:** 1: shedrun: tree record paths and a byte-level seed decoder
