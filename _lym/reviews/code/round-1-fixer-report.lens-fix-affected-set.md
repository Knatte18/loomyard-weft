```yaml
lens: fix-affected-set
outcome: reported
```

No findings. (Lens fix-affected-set: I reran the search for each finding whose premise is a set.)

- **F1 (fingerprint-mismatch phrase and `PlanEditError` callers):** the old "since the run state was created" text is gone from `internal`. The only call to `runplan.PlanEditError` is `internal/websterengine/runplan.go:155`, which now prepends "the plan changed since state.json was created; ". That matches `runlevel.go:575`. Every other caller, including the begin-batch, record-batch, recover-batch and validate paths, goes through the webster wrapper.
- **F3 (runplan sites that name an implementer's verbs or webster):** non-test runplan code names webster or a verb at `conflict.go:142` and `conflict.go:172` (the begin-batch header) and `postbatch.go:163` and `postbatch.go:166` (the handle-bind log). These are the only hits, and the updated `doc.go` names exactly them.
- **F5 (stale moved-helper names):** `changedPlanFiles`, `batchIdentity` and `batchCardIDs` no longer appear in any Go file under `internal`. Two hits remain under `docs/research/`, in `lens-fan-lab-phase0.md:78` and a `calls.tsv` capture. Both are historical research records, not live comments, so I count neither as a miss.
- **F2 (callers of `CheckRecoveredSuspects`):** I did not run a separate search. The earlier grep already showed `recoverbatch.go` as the only non-test caller of `runplan.CheckRecoveredSuspects`, and it now goes through `recoveredSuspectReasons`. `export_test.go` aliases the same function.
- **F4:** a single doc section, so no set to cover.

Tool calls:
1. Grep of `internal` for the moved helper names in non-test Go files, to find any stale symbol outside the cited lines.
2. Grep for the `runplan.PlanEditError` callers and both mismatch phrases, because my context held only the earlier grep from before the edit.
3. Grep of `internal/runplan` for webster and verb names, to cover F3's set.
4. Grep of `internal` and `docs` for the moved names including test files, to check beyond non-test Go code.
