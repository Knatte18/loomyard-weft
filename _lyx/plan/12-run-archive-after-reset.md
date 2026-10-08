# Card 12 — run-archive-after-reset

**Create:**
- `plan:internal/websterengine#ArchiveRunAfterReset` -> `func ArchiveRunAfterReset(engine shuttleengine.Engine, geom Geometry, st *State) ([]string, error)`

**Edit:**
- `internal/websterengine#Run`
- `internal/websterengine#freshPendingDrop`
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
- `internal/websterengine#checkSuspectPaths`
- `internal/websterengine#contractWritesFor`
- `internal/websterengine#splitContractPaths`
- `internal/websterengine#splitPlanPaths`

**Intent:** Give the reset the archive `run --fresh` performs, behind a pending-findings guard that never refuses toward a verb that cannot clear it, so card 14's `reset --to start` can end the run record (D3).
The decision record names `ArchiveRunRecord` for this; that function needs a told destination outside the webster dir and moves its whole content, which the reset has no destination for, so the reset uses the in-place archive `run --fresh` performs, which is the decision's intent ("the same archive `run --fresh` uses").

The archive: `Run`'s fresh branch's three steps (state.json renamed to `state-<stamp>.json`, the reports dir to `reports-<stamp>`, the rendered prompts cleared) move into one unexported function in `archive.go`, which `Run`'s fresh branch calls at the point it runs them today, so `Run --fresh` keeps its order (guard, new partition, archive) and its behavior.
`ArchiveRunAfterReset` is the reset's entry: it runs the pending-findings guard against the tree as it stands, then that archive, and returns the guard's drop warnings; a following plain `lyx webster run` finds no state and starts a new run.

Pending findings, per the shared decision `reset-guard-judges-against-head`: `freshPendingDrop`'s path checks, the uncleared-contract check, the suspect-path check over the evidence base and the plan-path check with its restorable-or-dropped split, move into one unexported function in `archive.go` that takes the evidence base and its caller's re-run step and returns the drop warnings or the refusal.
`freshPendingDrop` keeps everything before them, its `opts.Fresh` gate, the missing-commit refusal, `headBeforeEveryStart` and its HEAD-not-at-the-start refusal, then calls the shared function with the start commit and its own `run --fresh` texts, so every text `Run --fresh` returns stays as it is (card 17 rewords them).
The one reorder: `freshPendingDrop` now refuses on HEAD before the path checks, so a run whose HEAD is off the start and whose paths would also refuse gets the HEAD refusal first; the reset it names then reaches the path refusal through the reset's own guard.
`ArchiveRunAfterReset` calls the shared function with the worktree's HEAD as the evidence base and reads no recorded commit: after a moving reset HEAD is the start, so the checks are the ones `run --fresh` runs there, and on card 14's archive-only path HEAD is the tree the next run starts from, so a branch rewritten under the run never refuses through the guard.
It thereby refuses only where the caller can clear the state, each refusal naming the clearing step, then re-running `lyx webster reset --to start`:
- an uncleared contract file a fork wrote last: `rm <path>`;
- a plan path differing from the recorded plan: `lyx webster restore-plan`, or `lyx webster rebaseline --card NN` for an unbegun card;
- a suspect path differing from HEAD: `restoreStep`'s `git checkout <HEAD> -- <path>`, with its clause for a path HEAD does not hold.
Every other pending finding, and every batch record's `Uncheckable` entries, is dropped with the archived state under the warning `run --fresh` gives it.
Bound: on the reset the guard lets through exactly what D3's archive-only path lets through, a branch rewritten under the run, and drops a pending finding only once its paths match HEAD or no verb can check them; the dropped findings stay readable in the archived state, and a refusal leaves the run record in place.

`archive_test.go` gains `TestArchiveRunAfterReset`, over the package's `fakeGit`: a row per guard refusal (contract file, differing plan path, suspect path differing from HEAD), each asserting the reset caller's way forward and that `state.json` and the reports dir are left in place, and a rewritten-branch row, a recorded start that is missing and not an ancestor of HEAD with every pending path matching HEAD, asserting the record is archived with one drop warning per finding.
The rename and the prompt clearing stay pinned by `TestArchiveRunFile`, `TestArchiveReportsDir` and `TestRun_EntryHousekeeping`'s `--fresh` row, which `Run`'s unchanged call keeps green, and card 14's `TestResetCmd` step shows the reset's archive end to end.
`test-budget.yaml` raises `internal/websterengine` by one for `TestArchiveRunAfterReset`.
`refusal-spec.md` gains the `reset --to start: pending findings kept` row (the archive's pending-findings guard refuses; correctness halt; `rm <path>`, `lyx webster restore-plan` or `lyx webster rebaseline --card NN`, or `git checkout <HEAD> -- <path>` per condition, then re-run `lyx webster reset --to start`).
The package doc's archive paragraph names `ArchiveRunAfterReset`, its guard and why the guard judges against HEAD.

**ImpactSummary:** Adds one exported function whose callers are cards 14 and 15's reset; `Run --fresh` calls the factored archive and the factored path checks in the same places with unchanged behavior and texts.
