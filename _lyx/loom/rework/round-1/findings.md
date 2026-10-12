---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shedrun/seed.go:143
    summary: The PR keeps ParseSeedTolerant and the report calls it, breaking the parent decision that run-robustness's tolerant ParseSeed replaces it after Merge-In
    fix: Delete ParseSeedTolerant from internal/shedrun/seed.go. In internal/reportengine/runs.go:228 call shedrun.ParseSeed(seedBytes) instead. In internal/shedrun/seed_test.go drop the ParseSeedTolerant assertions added to TestParseSeed, including the [1] decode-error check. Remove the criterion that ReadSeed still rejects a seed with an unknown field from any test that pins it. Keep the encoding/json import in internal/alignment/records.go, which ParseAttemptDir needs.
---

### [BLOCKING] ParseSeedTolerant survives the parent decision that removes it

**Location:** `internal/shedrun/seed.go:143`, `internal/reportengine/runs.go:228`, `internal/shedrun/seed_test.go:144`

**Issue:** The decision record's entry `Added after Discussion (parent, 2026-10-12): Seed parsing follows run-robustness D4` says that after Merge-In the run drops `ParseSeedTolerant` and its "ReadSeed still rejects it" criterion and uses the tolerant `ParseSeed`.
The entry's rationale is that two parsers for one record contradict `PATTERN-tolerant-records`.
The entry was written at 16:02, and the Merge-In of `main` at `e5b56fe91` (run-robustness) is commit `7af4bf32f`, made at 16:22.
At `fd539e313`, `ParseSeedTolerant` is still in `shedrun`, `reportengine` still reads the seed through it, and `TestParseSeed` gains assertions on it.
Merge-Review round 1 raised that the two parsers now decode identically (F1).
The fixer then made `ParseSeedTolerant` call `decodeSeed` instead of deleting it, so the second seed parser the decision rules out shipped.
The only difference left between the two is that `ParseSeedTolerant` skips `ParseSeed`'s driver and params validation, and the decision does not ask for that.

## Scope and overlap

The rest of the diff matches the goal and D1 to D12: `lyx report convergence` in `internal/reportcli` and `internal/reportengine`, tolerant readers in the owning packages, the `haltnote` leaf, the `gitrepo` history and blame reads, and docs and PATTERN updates in the same change.
In `internal/alignment/records.go` the PR re-adds the `encoding/json` import for `ParseAttemptDir`, which follows the decision's "keeps internal/alignment/records.go compiling" clause.

`lyx loom peers` lists no landed parent commits.
Three peers are in flight:

- `operator-followups` changes `docs/overview.md`, which this run also changes.
  This run's change is only the module-tree lines and the module-list entry for `report`, which is a shared bookkeeping file and not an overlap.
  No other path is shared.
- `intake-bugs-1012` lists no changed paths yet, and its board entry does not touch the report.
- `cleanup-1012` lists no changed paths yet.
  Its tokencount cutoff-skew guard is in `tools/tokencount`, which this run does not change; it is related to this run's open risk on archive tag date skew, and neither contradicts the other.

`cmd/lyx/testdata/test-budget.yaml` gains rows for the new packages, which is bookkeeping.
`pattern/PATTERN-leaf-packages.md` gains a new index line for `haltnote`, which is also bookkeeping.
