# Card 11 — recover-failed-batch

**Edit:**
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/render.go`
- `contracts/stencils/webster/webster-prefix-recovery.md`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/template_test.go`

**Uses:**
- `internal/websterengine/batchfail.go`
- `internal/websterengine/recordbatch.go`

**Intent:**
Make `recover-batch NN` the working way forward from every failed batch, and hand the recovery strand the failure digest so it treats each named `parent-write` path as suspect (D6).

`recoverbatch.go`:

- `refuseRecoveringDoneReport` also passes when the prior record is terminal `failed`, beside the existing dead-orphan exception.
- `RecoverSpawnOrAttach` renders the prior record's failed digest, when there is one, as a failure-digest block — the reasons, then the suspect paths — and passes it to the recovery prompt.
- `PersistRecoveryTerminal`: an `ErrCardNotDone` from `postBatchChecks` goes through `failBatch` on the freshly loaded state (report archived, record terminal `failed`) and returns the `*BatchFailedError` with the pass's warnings, instead of leaving the recovery record non-terminal.
  The next `recover-batch NN` then spawns a fresh recovery strand rather than re-attaching to the finished one and failing the same checks forever.
  Update the doc comment's last paragraph.

`render.go`: `RenderRecoveryPrompt` gains a `failureDigest string` parameter after `prevDigest`, filled into a new optional marker `failure_digest` (`none` when empty), added to the `stencil.FillOptional` optional list so an older deployed stencil still renders.

`webster-prefix-recovery.md`: add a short section, "Why this batch is being recovered", holding `{{.failure_digest}}`.
It tells the strand that a non-`none` digest lists why webster rejected the batch's earlier report, that each suspect path is to be reverted or re-derived before the card is finished, and that it continues from the committed tree.
Name the marker in the header comment's marker list.

Tests:

- `recoverbatch_test.go` (integration): after card 8's fork-contract-write failure, card 8's failing-verify failure and card 10's failing done-check, each driven through `RecordBatch` on a real repo, `RecoverSpawnOrAttach` spawns (no refusal), the rendered prompt contains every reason and suspect path, and the new recovery record carries `Cards` (card 4).
  `PersistRecoveryTerminal` over an unresolved Create → record terminal `failed`, report archived, `errors.Is(err, ErrBatchFailed)`; a following `RecoverSpawnOrAttach` spawns a fresh strand.
  Update `TestRecoverBatch_TerminalRunsTheSamePostBatchChecksAsRecordBatch` to the terminal-failed outcome.
- `template_test.go`: every `RenderRecoveryPrompt` call gains the new argument; add one case asserting a non-empty failure digest renders verbatim and an empty one renders `none`.

**ImpactSummary:** recover-batch accepts failed batches and fails its own not-done recoveries terminally; RenderRecoveryPrompt's one production caller and its template tests change.
