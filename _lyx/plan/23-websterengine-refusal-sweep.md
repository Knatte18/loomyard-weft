# Card 23 — websterengine-refusal-sweep

**Edit:**
- `internal/websterengine/runlevel.go`
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/recordbatch_test.go`

**Intent:**
Close the remaining webster-engine refusals that name no way forward (D1), and give every non-grouped webster-engine row its reaching test.
Every added way forward is a trailing `way forward: …` clause, the overview's `way-forward-phrasing` shared decision, so tests and the refusal table (card 28) match on one form.

Messages to fix:

- `Run`, unapproved plan: approve the plan through its review, then re-run `lyx webster run`.
- `Run`, zero execution batches, and the validation refusal: fix the named cards in the plan, run `lyx webster rebaseline` when state.json already records this run, then re-run `lyx webster run`.
- `Run`, quarry unavailable: transient; re-run `lyx webster run` once quarry answers.
- `Run`, start-master failure: transient; re-run `lyx webster run`.
- `MasterAskingError`, `MasterDiedError`, `MasterTimeoutError` (`Error()` methods): re-run `lyx webster run` (re-step the Webster row); a fresh Master resumes from state.json.
- The run-exit refusals in `mapMasterDone`, `verifyEveryBatchDone`, `runExitAuditCrossCheck`'s nil-audit and audited-count checks, and `runIntegrationStage`'s missing integration report under `done`: re-run `lyx webster run`; a fresh Master resumes from state.json and re-drives every batch without a done record.
- A malformed `outcome.yaml` at run exit: re-run `lyx webster run`; the stale file is archived and a fresh Master writes a new one.
- A batch number not in the plan's batches: `lyx webster status` lists the run's batches; name one of those.
  The text lives in `findBatch` (`beginbatch.go`), so begin-batch, record-batch, await-batch and recover-batch all carry it; the test reaches it through `BeginBatch` and `RecordBatch`.
- `BeginBatch`, model-switch injection failure: transient; re-run `lyx webster begin-batch NN`.
- `RecordBatch`, a malformed batch report (`ParseReport` error): `lyx webster recover-batch NN` archives the malformed report and re-drives the batch (its `refuseRecoveringDoneReport` passes a report that does not parse).
- `PersistRecoveryTerminal`, no recorded state for the batch: re-run `lyx webster recover-batch NN`.
- `recoverSpawn`, recovery-strand start failure: transient; re-run `lyx webster recover-batch NN`.

Already carrying a way forward, and only needing a test that takes it where none exists: `ErrRunBusy`, the mid-merge refusal, the head-SHA mismatch and parent-merge refusals, begin-batch's report-exists refusal, recover-batch's OK-report refusal, record-batch's recovery-batch refusal.

Grouped rows, left as they are with no per-row test: nil-dependency preconditions (`Plan`/`State` nil, `ErrNilBatcher`), missing role model-specs, unrecognized shuttle outcome, config decode errors, render/template fill errors, and raw I/O failures.

Tests (integration, real scratch repo, state as the verbs write it): for each fixed row, reach the refusal, assert the message contains its way forward, take it (re-run `Run` with a fake Master that now finishes, approve the plan, fix and rebaseline the plan, call `RecoverSpawnOrAttach` over the malformed report), and assert the refused verb or the named verb then succeeds.
For each already-carrying row without a test, add one of the same shape.

**ImpactSummary:** Message text and tests only across webster's engine verbs; no control flow changes, and callers matching sentinels with errors.Is are unaffected.
