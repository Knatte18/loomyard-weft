# Card 2 — batcher estimator and segment cost

**Create:**
- `internal/batcher/estimate.go`
- `internal/batcher/estimate_test.go`
- `internal/batcher#SizeSource`
- `internal/batcher#Weights`
- `internal/batcher#SegmentCost`
- `internal/batcher#DiskSizes`
- `internal/batcher#ProfileWeights`

**Edit:**
- `internal/batcher/config.go`
- `internal/batcher/config_test.go`
- `internal/batcher/doc.go`
- `internal/planparser/targetdirs.go`
- `internal/planparser/targetdirs_test.go`

**Intent:**
The card-size estimator, its size source, the segment-cost function and the coefficients type with its parse (Decisions 4 and 5).
Nothing calls them in production yet; the calibration report (card 3) and the cost batchifier (card 5) do.

**planparser.**
Add `RefFile(plan *Plan, ref string) (string, bool)` beside `CardTargetDirs`: the worktree-relative file a card ref names — a path ref with a file extension is itself, a glyph or `plan:` handle whose `UnitPath` is a file is that file — and false for a directory or package unit and for an unmappable ref.
It shares `targetDir`'s internals (`parseGlyph`, `Glyph.UnitPath`, `hasFileExtension`), so glyphs map to files only through the glyph chokepoint and refs are classified only here.
`targetdirs_test.go` gets one table row per shape: a `.go` path, a file-unit glyph, a package-unit glyph, a handle, a directory path.

**Size source** (`estimate.go`).
`SizeSource` has two methods: `Lines(path string) (n int, exists bool, err error)` for a worktree-relative file, and `TestFiles(dir string) ([]string, error)` for the `*_test.go` files directly in a worktree-relative directory.
`DiskSizes(root)` implements it over the told absolute worktree root, deriving no path of its own; an absent file is `exists == false`, an absent directory has no test files.

**Weights.**
`Weights` has one float field per coefficient, each a `batcher.yaml` key under a profile's `weights:` map: `startup_context`, `fork_messages`, `target_messages`, `test_file_messages`, `uses_messages`, `context_per_line`, `package_context`.
`config.go`'s `config` type gains `Profiles map[string]profile` (`yaml:"profiles"`), a profile holding `Weights map[string]float64` (`yaml:"weights"`) for now; card 6 adds the other profile keys.
`ProfileWeights(baseDir, profile)` loads `batcher.yaml` through the same `configengine.LoadOrTemplate` call `Active` uses (one shared load helper, `PATTERN-config-strictness`) and returns the named profile's weights; it errors naming `batcher.yaml` when the profile is absent, a coefficient is missing, a coefficient is negative, or a key is not one of the seven.
`config_test.go` gains rows for each of those and for a valid profile.

**Estimate, per card** (Decision 4).
- Targets: one per target-group ref, except that a Rename pair counts once and is sized by its Old side.
- Files: each target and each `Uses` entry that `RefFile` maps to a file adds a read-set entry for that file weighing `lines × context_per_line`, when the file exists; a file that does not exist yet adds nothing.
- Directories: each distinct `CardTargetDirs` entry adds a package entry and a tests entry, each weighing `package_context`; the card's test-file count is the union of `TestFiles` over those directories.
- Messages: `target_messages × targets + test_file_messages × test files + uses_messages × len(Uses)`.
Read-set keys for a file, a package and a package's tests never collide, so two cards editing one file share more than two cards sharing only a directory.

**Segment cost** (Decision 5).
`SegmentCost` over cards `1..n` in order is `fork_messages × startup_context + Σ_k messages_k × (startup_context + W(R_1 ∪ … ∪ R_k))`, where `W` sums the weights of the distinct entries of the union.
A card's estimate is `SegmentCost` of its one-card segment.
Each coefficient enters once: the estimate applies the message, `context_per_line` and `package_context` coefficients, the segment cost `startup_context` and `fork_messages`.
A size-source error is returned wrapped.

**Tests** (`estimate_test.go`, untagged, on a fake size source and hand-built `planparser.Card` values):
a one-card cost equal to the hand-computed number;
two cards with identical read sets cost less merged than apart;
a small card followed by a card whose read set adds a large file costs more merged than apart;
a nonexistent target weighs nothing but still counts its messages;
test files counted once per distinct directory;
`DiskSizes` over a `t.TempDir()` tree: a file's line count, a missing file, a directory's test files only.

`doc.go` describes the estimator, the size source and the cost model.

**ImpactSummary:** additive: `internal/batcher` gains the estimator and `config` a `profiles:` field that existing `batcher.yaml` files omit, so `Active` behaves as before; `internal/planparser` gains one exported read-only function.
