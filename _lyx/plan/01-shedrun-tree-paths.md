# Card 1 — shedrun-tree-paths

**Create:**
- `plan:internal/shedrun#SeedTreeRel` -> `func SeedTreeRel(runID string) (string, error)`
- `plan:internal/shedrun#StatusTreeRel` -> `func StatusTreeRel(runID string) (string, error)`

**Rename:**
- `internal/shedrun#decodeSeed` -> `plan:internal/shedrun#DecodeSeed`

**Edit:**
- `internal/shedrun#TestPathConstructors`
- `internal/shedrun#TestRunIDVocabulary`
- `internal/shedrun#TestSeed_LegacyParentKey`
- `internal/shedrun/doc.go`

**Uses:**
- `internal/shedrun#ValidateRunID`
- `internal/shedrun#shedDirName`
- `internal/lyxdirs#LyxDirName`

**Intent:** Give a reader of a git tree the two run-record paths and the seed decoder it needs, so `tools/tokencount` never spells the `shed` segment, `seed.json` or `_lyx`, per `PATTERN-shed-run-directory` and `PATTERN-lyxdirs-single-declarer` (Decision 9).
`SeedTreeRel` and `StatusTreeRel` return the anchor-relative, slash-separated tree path of a run's `seed.json` and `status.json`: `path.Join` of `lyxdirs.LyxDirName`, `shedDirName`, the run id and the file name, in `paths.go` beside `SeedRel` and `StatusRel`.
They take no `*lyxcwd.Location`: a tree read from git has no disk to resolve an alias or the legacy `self` directory against, so the run id is joined as given, after `ValidateRunID` accepts it, and an id it refuses is returned as that error.
The path is slash-separated because `gitrepo.Repo.FileAtRevision` takes a slash-separated repo-relative path, where `SeedRel` uses `filepath.Join` for a fabric pathspec.
Their doc comments say both things: no alias, and the slash form.
Rename `decodeSeed` to `DecodeSeed` with an AST-aware rename, so the strict decoder, `DisallowUnknownFields` with the retired `parent` key accepted and dropped, is the one a tree reader calls; its doc comment opens with the new name and says it is the sole decoder of seed bytes, `ReadSeed` included, and that it applies no driver default and no validation, which `ReadSeed` keeps.
The package doc in `doc.go` names the tree-relative pair and `DecodeSeed` in the paragraph that describes the seed contract.
Tests, each in the existing test of the surface: `TestPathConstructors` gains a row per tree path asserting the exact slash-separated string `_lyx/shed/<id>/seed.json` and `_lyx/shed/<id>/status.json` for a plain id, and a row showing `self` is joined literally, not resolved to the worktree name; `TestRunIDVocabulary` asserts in its loop that `SeedTreeRel` and `StatusTreeRel` return an error exactly when `ValidateRunID` does, which is the refusal branch an invalid id reaches after no upstream check; `TestSeed_LegacyParentKey` gains a subtest calling `DecodeSeed` directly on the legacy-key bytes and on bytes with another unknown key, asserting the decoded seed and the refusal.
No new top-level test, so the package's row in `cmd/lyx/testdata/test-budget.yaml` stays.

**ImpactSummary:** Two new exported path functions and one unexported-to-exported rename whose only callers, `ReadSeed` and `WriteSeed`, are in `seed.go`; nothing outside `internal/shedrun` changes.
