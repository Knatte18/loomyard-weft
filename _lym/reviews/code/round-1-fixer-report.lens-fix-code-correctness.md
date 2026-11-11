```yaml
lens: fix-code-correctness
outcome: reported
```

**Findings**

1. **LOW** — `internal/adoptcli/worker_test.go`, row `ResumeRestartingAtReconcileReportsThePauseTheEarlierProcessSet`, for the `worker.go` change `if restart { record.Steps, record.StoppedPart = nil, nil }`.
   - The row passes even with the original defect restored.
   - After the first run stops at `owner`, the second run starts at `reconcile` with no restart. It re-runs `reconcile`, `pairs` and `pause`, and `recordStep` replaces each listed step. With the new `stepPause` loop over `inputs.Paused`, the re-run `pause` part carries `AdoptPaused`, `Run` and `PauseStanding` itself.
   - The envelope is therefore the same whether or not the earlier `Steps` were cleared. The row pins the `stepPause` change only.
   - Nothing pins keeping `Steps` on a resume. That matters only when the second process stops before it re-runs a step, for example a stop at `reconcile` or `pairs`. A row that stops the second run at `pairs` and asserts the earlier `pause` part is still in the envelope would pin it.

2. **LOW** — `internal/adoptcli/steps.go`, `stepDone`.
   - The `--keep-local` and `--drop-local` re-runs now carry the whole adopt's request, so both flags apply to every subject, not only the held-back projects.
   - This is needed for `SameSubjects` to match, and subjects already adopted no longer read as owned elsewhere, so they are not affected in practice.
   - A subject kept back by a non-stop error (`out.Error`) while its owner is still elsewhere would now also get `--drop-local`'s archive-and-reset. This is an edge case, and I found no test or doc that covers it.

**Checked, no defect found**
- `runWorker` resume path: `splitSubjects` on a record with `["alpha","root"]` gives `slugs=[alpha]` and `rootNamed=true`, the same split a fresh run records. Fresh and restart paths still replace `record`.
- `stepDone`: `run.Req` is already normalized, so `Command()` yields sorted slugs and flags that match `record.Request`. `slices` and `boardengine` are still used elsewhere in `steps.go`.
- `HeldBackStop…` row: with the old `stepDone` the command is `lym adopt alpha --keep-local`, `judgeStanding` refuses it, and the row fails.
- `StoppedStepPartLeavesTheRecord…` row: fails if `recordStep` stops clearing `StoppedPart`.
- `ResumeOfAnAllAdopt…` row: fails if the resume re-resolves `--all` against the board, because `gamma` would reach the owner write.
- `TestLiftOwnerPause…`: with a filter that matched adopt or hold entries, `HubDeps(nil, …)` would be reached and the test would fail.
- The `adopt_test.go` `wantErr` change to `lym adopt alpha beta --keep-local` matches the new command.
- The gate run reports tier 0 and tier 1 green.

**Tool calls:** none. Everything I needed was already in context.
