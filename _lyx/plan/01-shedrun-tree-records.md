# Card 1 — shedrun-tree-records

**Create:**
- `plan:internal/shedrun#TreeSeedRel` -> `func TreeSeedRel(runID string) (string, error)`
- `plan:internal/shedrun#TreeStatusRel` -> `func TreeStatusRel(runID string) (string, error)`
- `plan:internal/shedrun#DecodeSeed` -> `func DecodeSeed(data []byte) (Seed, error)`
- `plan:internal/shedrun#TestTreeRecordRels` -> `func TestTreeRecordRels(t *testing.T)`
- `plan:internal/shedrun#TestDecodeSeed` -> `func TestDecodeSeed(t *testing.T)`

**Edit:**
- `internal/shedrun#ReadSeed`
- `internal/shedrun/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedrun#ValidateRunID`
- `internal/shedrun#decodeSeed`
- `internal/shedrun#SeedRel`
- `internal/shedrun#StatusRel`

**Intent:** Give `internal/shedrun` the read side a tree read from git needs, so `tools/tokencount` can read a run's archived `seed.json` and `status.json` without naming the `shed` segment or `seed.json` and without decoding `Seed` itself (`PATTERN-shed-run-directory`).

`TreeSeedRel` and `TreeStatusRel` live in `paths.go`.
Each validates `runID` with `ValidateRunID` and returns its error unchanged on refusal;
on success it returns the slash-separated path `path.Join(lyxdirs.LyxDirName, shedDirName, runID, "seed.json")` (resp. `"status.json"`), the form `gitrepo`'s `FileAtRevision` takes.
They take no `*lyxcwd.Location`: no `self` alias resolution and no legacy `self`-directory fallback apply, since a tree in git has no disk to probe.
Their doc comments say so, and that they exist for reading a run's records out of a commit's tree, where `SeedRel` and `StatusRel` are for fabric commit pathspecs against the disk.

`DecodeSeed` lives in `seed.go`: it decodes `data` with `decodeSeed` (strict, unknown key refused, the retired `parent` key accepted and dropped), defaults an empty `Driver` to `DriverGo`, then validates the driver with `ValidateDriver` and refuses an empty `Params` key, each with an error naming what failed.
`ReadSeed` keeps its run-id validation and file read, and replaces its own decode, default and validation steps with one call to `DecodeSeed`, wrapping its error as `shedrun: decode seed for run %q: %w`.
A grep for `decode seed for run`, `seed for run` and `params key must not be empty` finds no test asserting the old texts, so no other test joins this card.

`doc.go`: the closing paragraph's claim that every constructor joins onto a told `*lyxcwd.Location`'s `AnchorPath()` gains the exception: `RunsRootRel`, `TreeSeedRel` and `TreeStatusRel` take no location, the last two because they address a tree read from git.
The paragraph naming `ReadSeed`/`WriteSeed` as the seed's sole reader and writer names `DecodeSeed` as the decoder over bytes from a git tree.

Tests, in `paths_test.go` and `seed_test.go`:
- `TestTreeRecordRels` pins both paths for a plain slug (`_lyx/shed/<slug>/seed.json` and `.../status.json`, slash-separated on every platform), pins that the `self` run id stays the literal `self` segment, and refuses `""`, `a/b`, `a\b`, `.` and `..` with an error and an empty path.
- `TestDecodeSeed` is table-driven: a full seed with `started_at` decodes to its fields; an absent driver decodes as `go`; a legacy `parent` key decodes and is dropped; an unknown key, an unknown driver and an empty params key are each refused; malformed JSON is refused.

`cmd/lyx/testdata/test-budget.yaml`: raise the `internal/shedrun` row by the two tests this card adds.

**ImpactSummary:** `ReadSeed`'s decode path moves into `DecodeSeed` with its behavior unchanged; every `ReadSeed` caller sees identical results, and only the wrapped text of a decode or validation error changes.

**Commit:** 1: shedrun: tree-relative seed and status paths and DecodeSeed
