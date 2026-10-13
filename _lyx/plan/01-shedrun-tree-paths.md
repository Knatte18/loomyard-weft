# Card 1 — shedrun-tree-paths

**Create:**
- `plan:internal/shedrun#TreeSeedRel` -> `func TreeSeedRel(runID string) (string, error)`
- `plan:internal/shedrun#TreeStatusRel` -> `func TreeStatusRel(runID string) (string, error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun#TestPathConstructors`
- `internal/shedrun#TestReadWriteSeed_InvalidResolvedIDRefused`
- `internal/shedrun#TestSeed_LegacyParentKey`
- `internal/shedrun/doc.go`

**Intent:**
Give a reader of a run tree read from git (Decision 9) the paths of a run's `seed.json` and `status.json` and a decoder over seed bytes, so `tools/tokencount` names neither the `shed` segment nor `seed.json` (`PATTERN-shed-run-directory`) nor `_lyx` (`PATTERN-lyxdirs-single-declarer`).

`TreeSeedRel` and `TreeStatusRel` go in a new file `internal/shedrun/treepaths.go`.
Each validates `runID` with `ValidateRunID` and returns its error unchanged on refusal;
otherwise it returns the slash-separated `path.Join(lyxdirs.LyxDirName, shedDirName, runID, "seed.json")` (respectively `"status.json"`), the repo-relative form `gitrepo.Repo.FileAtRevision` takes.
They take no `*lyxcwd.Location`, apply no `ResolveRunID` alias and no legacy `self` fallback: `TreeSeedRel("self")` names `_lyx/shed/self/seed.json`.
The file's header comment says why they exist beside `SeedRel`/`StatusRel`: those resolve against the disk, which a tree read at a revision has none of.

`DecodeSeed` goes in `seed.go` and holds the decoding `ReadSeed` does after its read: the strict `decodeSeed`, an absent or empty `Driver` defaulted to `DriverGo`, `ValidateDriver`, and the non-empty `Params` key check, each error as `ReadSeed` words it today minus the run id.
`ReadSeed` keeps its run-id validation, its file read and its not-found answer, then calls `DecodeSeed` and wraps its error with the run id as it does now, so its observable errors keep their prefixes.
`doc.go` gains a paragraph: the tree-path pair joins the same segment the on-disk constructors join, onto no anchor, for a caller reading a run's records at a git revision, and `DecodeSeed` decodes seed bytes such a caller read.
Its closing sentence, "Every constructor in this package is a plain filepath.Join onto a told *lyxcwd.Location's AnchorPath()", is narrowed to every constructor but the tree-path pair.

Tests, no new top-level test (the package is at its `cmd/lyx/testdata/test-budget.yaml` row):
- `TestPathConstructors` gains rows `TreeSeedRel` and `TreeStatusRel` whose `got` calls the function with `"self"` and drops the nil error, and whose `want` is `path.Join("_lyx", "shed", "self", "seed.json")` (respectively `status.json`), pinning both the slash layout and the absent alias.
- `TestReadWriteSeed_InvalidResolvedIDRefused` asserts `TreeSeedRel("../x")` and `TreeStatusRel("a/b")` each return an error and an empty path.
- `TestSeed_LegacyParentKey` gains one subtest pinning the exported `DecodeSeed`: `{"recipe":"loom"}` decodes with `Driver` `DriverGo`; the legacy key and unknown-key cases stay with the existing `ReadSeed` subtests, which now run through `DecodeSeed`.

**ImpactSummary:** `ReadSeed` keeps its signature and error prefixes; its callers across shed, loom and batten see no change, and the new exports have no caller until card 3.
