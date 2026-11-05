# Card 14 — drift-check-verb

**Create:**
- `plan:internal/driftcli#checkCmd` -> `func checkCmd() *cobra.Command`
- `plan:internal/driftcli#worktree.checkDeps` -> `func (w *worktree) checkDeps() (driftengine.Deps, error)`
- `plan:internal/driftcli_test#TestCheck_PrintsTheFindings` -> `func TestCheck_PrintsTheFindings(t *testing.T)`

**Edit:**
- `internal/driftcli#Command`
- `internal/driftcli/doc.go`

**Uses:**
- `plan:internal/driftengine#Check`
- `plan:internal/driftengine#CheckReport.Findings`
- `plan:internal/fabricengine#FindTrailerCommit`
- `plan:internal/fabricengine#AnchoredFileAt`
- `internal/shedrun#ReadSeed`
- `internal/loomrecipe#RowNames`
- `internal/driftcli_test#lymRun`

**Intent:** D8's `lym drift check`: any session in the run's worktree, such as a Webster card fork, runs the drift gate's check on demand.
The card follows the Drift-Expect rows (card 13), since the check reads the run's real recipe rows and judges a run before Drift-Expect as not yet sealed.

`checkDeps` builds the `driftengine.Deps` the check reads over the cwd's worktree: paths, config, the run status, `TrailerCommit` and `FileAt` over the worktree's location, `CaptureCurrent` over `captureRequest` with `InheritedSlot` read from the environment's `gateslot.InheritEnv`, `Compare` over `compareRequest` with no skip list, and `Rows` from `loomrecipe.RowNames` of the recipe `shedrun.ReadSeed` reads for the self run.
`checkCmd` (`check`, role audience, no arguments) runs `driftengine.Check` and prints, under `check`, whether the drift gate would pass, the unexpected cells, the faults and `Findings()`; before Drift-Expect it prints that no expectation is sealed yet and checks nothing (D6).
It writes no tracked record: only its scratch capture directory (D8).
Bound: it decides nothing; Drift-Verify repeats the check.

`TestCheck_PrintsTheFindings` (tier1, the package's hub fixture and `lymRun` helper, a capture command writing fixed files) seeds, as the anchor test seeds its run: a darn seed, so `Rows` is darn's order; a run status at Drift-Verify, past Drift-Expect; a scope record with scope `full`, `awaited_at` set and `diff: lines`; a passed `capture.json`; and a baseline directory holding a case file; with no seal commit and no `expected.yaml`, the expectation is empty.
It covers: the capture command writing a changed value prints it with case, identity, old and new value; the same run with the status at Drift-Await, before Drift-Expect, prints that nothing is sealed yet and runs no capture; the run with the baseline directory removed reports the missing baseline naming `lym drift defer` without failing and runs no capture; no tracked file changes.
The package doc lists `check`.

**ImpactSummary:** Adds one verb to the `lym drift` tree and one wiring method; existing verbs are untouched.
