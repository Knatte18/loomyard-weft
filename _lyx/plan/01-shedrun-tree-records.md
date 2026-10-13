# Card 1 — shedrun-tree-records

**Create:**
- `plan:internal/shedrun#TreeSeedRel` -> `func TreeSeedRel(runID string) (string, error)`
- `plan:internal/shedrun#TreeStatusRel` -> `func TreeStatusRel(runID string) (string, error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun#TestRunsRootRel`
- `internal/shedrun#TestSeed_LegacyParentKey`
- `internal/shedrun/doc.go`

**Intent:**
Give `internal/shedrun` the read side a tree read from git needs (Decision 9), so `tools/tokencount` can name and decode an archived run's `seed.json` and `status.json` without spelling the `shed` segment, `seed.json` or `_lyx` (`PATTERN-shed-run-directory`, `PATTERN-lyxdirs-single-declarer`).

`TreeSeedRel` and `TreeStatusRel` live in a new file `internal/shedrun/treepaths.go`, whose header says they name a run's records in a tree read from git, joined with `path.Join` onto no `*lyxcwd.Location`; `paths.go`'s header, which says each of its constructors is a `filepath.Join` onto a told location's `AnchorPath()`, stays true of that file.
Each validates `runID` with `ValidateRunID` and returns its error unchanged, then returns the slash-separated, repository-relative tree path `path.Join(lyxdirs.LyxDirName, shedDirName, runID, "seed.json")` (and `"status.json"`), the form `gitrepo.Repo.FileAtRevision` takes.
They take no `*lyxcwd.Location`: no `ResolveRunID` alias and no legacy `self` fallback applies, so `self` names the literal `self` directory.

`DecodeSeed` lives in `seed.go`.
It is the strict decode `ReadSeed` applies today, moved behind an exported name: `decodeSeed` (unknown key refused, the retired `parent` key accepted and dropped), then an absent or empty `Driver` defaulted to `DriverGo`, `ValidateDriver`, and the empty-`Params`-key check.
Its errors carry no run id: `shedrun: decode seed: <err>`, the `ValidateDriver` error, and `shedrun: seed params key must not be empty`.
`ReadSeed` keeps its run-id validation and its absent-file `(Seed{}, false, nil)`, and replaces its own decode-default-validate block with one `DecodeSeed` call whose error it wraps as `shedrun: seed for run %q: %w`.
A grep of the repository for `decode seed for run` and `params key must not be empty` finds no assertion outside `seed.go`, so the reworded `ReadSeed` errors break no test.
`WriteSeed` keeps calling the unexported `decodeSeed`, since it compares an existing seed without validating it.
`seed.go`'s file comment, which names `ReadSeed`, `WriteSeed` and `List` as the file's sole parser and writer, gains `DecodeSeed` among them.

Tests:
- `TestRunsRootRel` gains rows: `TreeSeedRel("x")` is `_lyx/shed/x/seed.json` and `TreeStatusRel("x")` is `_lyx/shed/x/status.json`, exact strings with forward slashes; `TreeSeedRel("self")` is `_lyx/shed/self/seed.json`, pinning that no alias applies; and `"a/b"` returns a non-nil error and an empty path from both functions, pinning the delegation to `ValidateRunID`, whose vocabulary `TestRunIDVocabulary` already covers.
- `TestSeed_LegacyParentKey` gains a subtest "DecodeSeed decodes as ReadSeed does", covering what the existing `ReadSeed` subtests and `TestReadSeed_AbsentDriverDefaultsToGo` do not: `{"recipe":"loom","started_at":"2026-01-02T03:04:05Z"}` decodes through the exported name with `Driver` `DriverGo` and `StartedAt` kept; `{"recipe":"loom","driver":"rust"}` and `{"recipe":"loom","params":{"":"x"}}` each return a non-nil error, the second reaching the empty-key check.

`doc.go` gains one paragraph, one sentence per line: the tree pair names a run's records in a tree read from git, validated but unaliased, and `DecodeSeed` is the decode `ReadSeed` applies, for seed bytes read from git.
Its closing paragraph, which says every constructor in the package is a `filepath.Join` onto a told location's `AnchorPath()`, is reworded to except the tree pair, which joins onto no location.

**ImpactSummary:** Additive within `internal/shedrun`: `ReadSeed`'s only change is its decode error prefix, which no test or caller matches, and `WriteSeed` is untouched.

**Commit:** 1: shedrun: tree paths and a byte decoder for archived run records
