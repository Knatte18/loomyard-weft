# Card 12 — run-archive-after-reset

**Create:**
- `plan:internal/websterengine#ArchiveRunAfterReset` -> `func ArchiveRunAfterReset(engine shuttleengine.Engine, geom Geometry, st *State) ([]string, error)`

**Edit:**
- `internal/websterengine#Run`
- `internal/websterengine#freshPendingDrop`
- `internal/websterengine#headBeforeEveryStart`
- `internal/websterengine/archive.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/archive_test.go`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/websterengine#archiveStateFile`
- `internal/websterengine#archiveReportsDir`
- `internal/websterengine#clearRenderedPrompts`
- `internal/websterengine#restoreStep`

**Intent:** Give the reset the archive `run --fresh` performs, behind the same pending-findings guard, so card 13's `reset --to start` can end the run record (D3).
The decision record names `ArchiveRunRecord` for this; that function needs a told destination outside the webster dir and moves its whole content, which the reset has no destination for, so the reset uses the in-place archive `run --fresh` performs, which is the decision's intent ("the same archive `run --fresh` uses").

The archive: `Run`'s fresh branch's three steps (state.json renamed to `state-<stamp>.json`, the reports dir to `reports-<stamp>`, the rendered prompts cleared) move into one unexported function in `archive.go`, which `Run`'s fresh branch calls at the point it runs them today, so `Run --fresh` keeps its order (guard, new partition, archive) and its behavior.
`ArchiveRunAfterReset` is the reset's entry: it runs the pending-findings guard against the tree as it stands, then that archive, and returns the guard's drop warnings; a following plain `lyx webster run` finds no state and starts a new run.

Pending findings: `run --fresh` drops `State.PendingAuditFindings` and every batch record's `Uncheckable` entries only behind `freshPendingDrop`'s evidence checks, and `ArchiveRunAfterReset` keeps that guard, so no reset to start drops a finding `run --fresh` would refuse to drop.
`freshPendingDrop` and `headBeforeEveryStart` take their caller's re-run steps in place of the hard-coded `run --fresh` texts (`freshPendingDrop`'s `opts.Fresh` gate stays with the `Run` caller).
For `Run` every text stays as it is (card 16 rewords them); for the reset each refusal names a step other than the reset itself, then re-running `lyx webster reset --to start`:
- an uncleared contract file a fork wrote last: `rm <path>`;
- a plan path differing from the recorded plan: `lyx webster restore-plan`, or `lyx webster rebaseline --card NN` for an unbegun card;
- a suspect path differing from the start commit: `restoreStep`'s `git checkout <start> -- <path>`, with its clause for a path the start does not hold;
- HEAD not at the start commit, and `headBeforeEveryStart`'s recorded starts that share no single oldest commit while HEAD is not an ancestor of every one: `lyx webster accept-audit`, or `lyx webster accept-audit --batch NN` for an uncheckable record, which clears a finding on evidence.
A recorded commit missing from the repository keeps `missingCommitsClause`'s fetch-then-re-run step, which already names no reset.
Bound: the guard only adds refusals; what it lets through is exactly what `run --fresh` lets through today, and a refusal leaves the run record in place.

`archive_test.go` gains `TestArchiveRunAfterReset`, over the package's `fakeGit`: a row per guard refusal (contract file, differing plan path, differing suspect path, HEAD not at the start, starts with no single oldest commit and HEAD not before each), each asserting the reset caller's way forward and that `state.json` and the reports dir are left in place.
The rename and the prompt clearing stay pinned by `TestArchiveRunFile`, `TestArchiveReportsDir` and `TestRun_EntryHousekeeping`'s `--fresh` row, which `Run`'s unchanged call keeps green, and card 13's `TestResetCmd` step shows the reset's archive end to end.
`test-budget.yaml` raises `internal/websterengine` by one for `TestArchiveRunAfterReset`.
`refusal-spec.md` gains the `reset --to start: pending findings kept` row (the archive's pending-findings guard refuses; correctness halt; the per-condition step above, then re-run `lyx webster reset --to start`).
The package doc's archive paragraph names `ArchiveRunAfterReset` and its guard.

**ImpactSummary:** Adds one exported function whose one caller is card 13's reset; `Run --fresh` calls the factored archive in the same place with unchanged behavior, and the guard's texts for it are unchanged.
