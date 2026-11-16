# Move Webster's batch machinery and run-plan state into internal/runplan

Webster's batch machinery moves into a new package, `internal/runplan`, so a second implementer can import it instead of copying it (D11).
Webster's behaviour does not change.

## What moved

- **Run-plan state.** `runplan.State` holds the plan fingerprint, the plan file hashes, the overview frame hash, the recorded partition and the batch records.
  `websterengine.State` embeds it beside its Master, audit and fixer fields.
  `encoding/json` flattens the embedded struct, so `state.json` keeps the same keys.
  `BatchState` moves whole. The fields only Webster fills (fork transcripts, audit warnings, recovery records) are `omitempty`, so another implementer leaves them empty.
- **Report contract.** `Report` and `ParseReport` gain an optional `journal` key. Webster's forks leave it empty.
- **Plan fingerprint and re-baseline.** This covers the fingerprint, the file and frame hashes, the plan-baseline store, the edit checks and `RestampFingerprint`.
- **Plan conflict.** This covers `PlanConflictError`, the conflict record and its route claim, and the dispatch re-resolution that `BeginBatch` ran inline, now `runplan.ResolveAtDispatch`.
- **Post-batch checks.** `runplan.PostBatchChecks` runs the done checks, the re-sign rebind, handle binding, the scope guard and drift repair.
  It takes the recorded partition mapped onto the current plan and works out the completed and unbegun cards itself.
  The old `Completed` input is gone.
  It records a later card's drift warning through a `RecordWarning` callback, so Webster's audit ledger stays in Webster.
- **Other moved pieces.** These are the suspect-path checks and their path helpers, the digest and `Distill`, card gate rendering, `AwaitBatch`, the recovery-strand stop and the verify-failure record (`VerifyGateReport`, the parser, the card hint and the findings renderer).

## How the pieces are told their context

`runplan.Geometry` names the implementer's worktree, pane directory, plan directory, run, reports, prompts and scratch directories, Git and code index.
Webster builds it from its own `Geometry` with the unexported `runPlan()`, and its run directory is `WebsterDir`.
`runplan.Git` holds the subset of git questions the shared code asks, and `websterengine.Git` embeds it.

Every shared refusal that ends in a way forward takes that text from its caller:
the plan conflict, the fingerprint mismatch, the partition mismatch, an unknown batch, the later-target hand-back and a suspect plan path.
Webster's text lives in `internal/websterengine/runplan.go` and matches the text it rendered before.

## Compatibility

`internal/websterengine/runplan.go` keeps every moved exported name as a type alias, a constant or a function variable, so Webster's importers compile without edits.
Exported Webster functions that take `*websterengine.State` stay as Webster functions that pass `&st.State`: `PlanEditError`, `DispatchScope`, `RecordPartition`, `RestampPlanBaseline` and `AwaitBatch`.

A composite literal cannot set a promoted field, so every test literal of `websterengine.State` that set a run-plan field now nests it under `State: runplan.State{...}`.
Those edits are mechanical, in the tests of websterengine, webstercli, loomcli, loomshed, failuretrace and tools/tokencount.
Webster's internal tests reach the moved unexported helpers through `export_test.go`.
`TestMoveBegunCardHashes` moved to `internal/runplan`, since the function it pins is unexported there.
A new runplan test covers the optional `journal` key.

## Design decisions

- The shared sentinels now read `runplan: ...` instead of `webster: ...`, because Weaver will return them too.
  The pinned `webster: handle bind` log line and the on-disk conflict record header keep their text.
- `cmd/lym`'s test-only allowlist drops `websterengine.Fingerprint` and `websterengine.WriteReport`.
  Each is now a variable whose `runplan` target production code reaches.
- `runplan.State` holds the five pieces the record names. `CurrentBatch` stays in Webster's state.
