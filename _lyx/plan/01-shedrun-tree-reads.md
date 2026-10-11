# Card 1 — shedrun-tree-reads

**Create:**
- `plan:internal/shedrun#TreeSeedRel` -> `func TreeSeedRel(runID string) (string, error)`
- `plan:internal/shedrun#TreeStatusRel` -> `func TreeStatusRel(runID string) (string, error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun/doc.go`
- `internal/shedrun#TestPathConstructors`
- `internal/shedrun#TestReadWriteSeed_InvalidResolvedIDRefused`
- `internal/shedrun#TestSeed_LegacyParentKey`

**Intent:**
Give `internal/shedrun` the reads a tree taken from git needs, so `tools/tokencount` can locate and decode an archived run's records without naming the `shed` segment or `seed.json` itself (Decision 9, `PATTERN-shed-run-directory`).

`TreeSeedRel` and `TreeStatusRel` go in `paths.go`, beside `SeedRel` and `StatusRel`.
Each validates `runID` with `ValidateRunID` first and returns its error unchanged, then returns the slash-separated tree path `path.Join(lyxdirs.LyxDirName, shedDirName, runID, "seed.json")` (and `"status.json"`).
They take no `*lyxcwd.Location` and apply no alias and no legacy `self` fallback: `"self"` joins the literal `self` segment, since a git tree holds no worktree to resolve it against.
They join with `path`, not `filepath`, because `gitrepo`'s `FileAtRevision` takes slash-separated repo-relative paths.

`DecodeSeed` goes in `seed.go`.
It runs the strict `decodeSeed` (an unknown key is an error, the retired `parent` key is accepted and dropped), defaults an empty `Driver` to `DriverGo`, validates the driver with `ValidateDriver`, and refuses an empty `Params` key.
Its driver and params errors read `shedrun: seed: <reason>`, with no run id, since it is handed bytes alone.
`ReadSeed` keeps its signature and calls `DecodeSeed` on the bytes it read, wrapping every `DecodeSeed` error as `shedrun: decode seed for run %q: %w`, so the on-disk reader and the tree reader decode identically.
That folds `ReadSeed`'s former `shedrun: seed for run %q:` driver and params prefix into the decode prefix; no test or caller matches either text (a repository grep for both finds only `seed.go`).
`WriteSeed` keeps calling the unexported `decodeSeed` for its agreement check, unchanged.

`doc.go` gains a paragraph naming the tree reads, and its closing paragraph, which says every constructor is a `filepath.Join` onto a told `*lyxcwd.Location`, now excepts `TreeSeedRel` and `TreeStatusRel` as the tree-path pair that takes no location.
The file header of `paths.go` makes the same exception, and the file header of `seed.go` names `DecodeSeed` beside `ReadSeed`, `WriteSeed` and `List` as the file's seed parsers and writer.

Tests, all as rows or assertions of existing tests:
- `TestPathConstructors` gains rows `TreeSeedRel` and `TreeStatusRel` for run-id `"self"`, wanting `path.Join("_lyx", "shed", "self", "seed.json")` and the `status.json` twin.
  These rows sit beside the existing `SeedRel` row, which wants the worktree slug for the same `"self"`, so the pair pins that the tree reads apply no alias.
  A row's `got` closure returns the error's text in place of the path when the call fails, so a wrongly refused `"self"` fails the comparison.
- `TestReadWriteSeed_InvalidResolvedIDRefused` gains assertions that `TreeSeedRel("a/b")`, `TreeStatusRel("..")` and `TreeSeedRel("")` each return an error.
- `TestSeed_LegacyParentKey` gains a subtest "DecodeSeed decodes as ReadSeed does": `DecodeSeed` of `{"recipe":"loom","parent":"ab:hub","started_at":"2026-01-02T03:04:05Z"}` returns recipe loom, driver `go` and the given `StartedAt`, and `DecodeSeed` of the same bytes plus `"bogus":"x"` returns an error.

**ImpactSummary:** Adds three exported functions to `internal/shedrun`; `ReadSeed`'s five production callers see no change in signature or result, and only its driver and params error prefix changes, which none of them matches.
