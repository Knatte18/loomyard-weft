# Card 8 — drift-seal-baseline-checks

**Extends:**
- `internal/driftengine#Deps`
- `internal/driftengine/doc.go`

**Edit:**
- `internal/loomcli#driftDeps`
- `internal/loomcli#driftTables`
- `internal/loomcli#loomCLI.wire`
- `internal/testkit/envkit#driftDeps`

**Create:**
- `plan:internal/driftengine#SealTrailerKey` -> `const SealTrailerKey = "Drift-Seal"`
- `plan:internal/driftengine#BaselineTrailerKey` -> `const BaselineTrailerKey = "Drift-Baseline"`
- `plan:internal/driftengine#DeferHaltPrefix` -> `const DeferHaltPrefix = "drift defer halt: "`
- `plan:internal/driftengine#BaselineDigest` -> `func BaselineDigest(dir string) (string, error)`
- `plan:internal/driftengine#BaselineState` -> `type BaselineState struct`
- `plan:internal/driftengine#CheckBaseline` -> `func CheckBaseline(d Deps) (BaselineState, error)`
- `plan:internal/driftengine#SealExpectation` -> `func SealExpectation(d Deps, extra ...fabricengine.Trailer) error`
- `plan:internal/driftengine#SealState` -> `type SealState struct`
- `plan:internal/driftengine#CheckSeal` -> `func CheckSeal(d Deps, rec Records, baselineHead string) (SealState, error)`
- `plan:internal/driftengine#PastRow` -> `func PastRow(rows []string, current, row string) bool`
- `plan:internal/driftengine#TestSealAndBaseline` -> `func TestSealAndBaseline(t *testing.T)`

**Uses:**
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/loomrecipe#RowNames`
- `plan:internal/driftengine#ReadRecords`
- `plan:internal/driftengine#Records`
- `plan:internal/driftengine#commitStore`
- `plan:internal/driftengine#commitStore.deps`

**Intent:** The seal (D6) and the baseline digest (D7) as code checks every later check runs first; no row calls them yet.

- `Deps` gains `Rows []string`, the run's recipe row order.
  `loomcli`'s `wire` reads it through `loomrecipe.RowNames` for the run's recipe and passes it on `driftTables`, which gains `rows`; `envkit`'s `driftDeps` fills the darn order.
  `PastRow` reports whether `current` comes after `row` in `rows`; a row absent from `rows` is not past.
- `BaselineDigest` hashes the baseline directory's tree, every regular file's slash-separated relative path and content in path order, with sha256; a missing directory is an error wrapping `fs.ErrNotExist`.
- `CheckBaseline` finds the oldest weft commit carrying `Drift-Baseline` through `TrailerCommit` and recomputes the digest of `Paths.BaselineDir`.
  `DeferHaltPrefix` marks every reason, here and in the later rows' Stucks, whose way forward is `lym drift defer`, right after the leading `shedadapters.DecisionHaltPrefix` a row's Stuck opens with (Shared Decisions); `Defer` (card 19) admits a halt by it.
  `BaselineState` holds `Fault`, a reason carrying `DeferHaltPrefix` and naming the parent's `lym drift defer` as the way forward, set for a missing baseline directory and for a digest that differs from the trailer's, and `Unguarded`, true when no commit carries the trailer, which checks nothing (a capture counted before this lands and already past Drift-Expect, D7).
- `SealExpectation` reads `Paths.Expected`, hashes it and commits it alone through `CommitWithTrailers` with `Drift-Seal: <sha256>` and any extra trailer, the `Drift-Baseline` one for a counted capture with no digest yet (D7).
- `CheckSeal` decides the expectation containment reads, in D6's order, and returns `SealState` (`Sealed` bytes, `Empty`, `NotYet`, `Fault` naming the failed check and its way forward):
  - a `discard` amendment in `rec.Scope.Amendments`: no check, `Empty`;
  - `rec.Scope.Unsealed`: `Empty`, and any seal commit is ignored;
  - the current row not past Drift-Expect by `PastRow(d.Rows, status row, "Drift-Expect")`: `NotYet`;
  - no seal commit and no `expected.yaml`: `Empty`;
  - an `expected.yaml` with no seal commit: `Fault` naming `lym drift amend <slug> --discard`;
  - otherwise the oldest commit carrying `Drift-Seal` is the seal: its copy, read through `FileAt`, must hash to the trailer value, else `Fault` naming `--discard`; its `WarpSHA` must equal `baselineHead`, else `Fault` naming `--discard`; the current file must hash to the trailer value, else `Fault` naming `--restore` with `Sealed` still the seal's copy, so a gate or verb that reports without failing checks against it (D6).
  The status row is read through `Deps.Status`.

`TestSealAndBaseline` (tier0, `commitStore` and a `t.TempDir` baseline, `t.Parallel`) covers, as rows: a valid seal reads its bytes; an edited `expected.yaml` faults naming `--restore` and keeps the sealed copy; a seal whose `Warp-SHA` differs from the baseline head faults naming `--discard`; a seal copy that fails its own hash faults naming `--discard`; a file with no seal past Drift-Expect faults naming `--discard`; no file and no seal past Drift-Expect is `Empty`; a row before Drift-Expect is `NotYet`; an `Unsealed` record ignores a later seal; a `discard` amendment skips the check; an edited baseline file and a missing baseline directory each fault naming `lym drift defer`; no `Drift-Baseline` commit is `Unguarded`.
The edited-baseline row is the one that shows the recorded digest is read from the commit, not recomputed from the worktree.
The doc.go gains a `# Seal` section.

**ImpactSummary:** Adds checks no row calls yet and one Deps field, wired in loomcli from the recipe's row order and faked in envkit; no behavior changes.
