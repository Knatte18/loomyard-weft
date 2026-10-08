# Card 20 — amended-cards-recovery

**Edit:**
- `internal/websterengine#failBatch`
- `internal/websterengine#BatchFailedError`
- `internal/websterengine#RecordBatch`
- `internal/websterengine#PersistRecoveryTerminal`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine#failureDigestBlock`
- `internal/websterengine/doc.go`
- `internal/websterengine/batchfail_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/webstercli#websterCLI.recordBatchCmd`
- `internal/webstercli#websterCLI.recoverBatchCmd`
- `internal/webstercli/verbs_test.go`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/claims_test.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `plan:internal/websterengine#AmendedCard`

**Intent:** An amendment recorded against the in-flight batch (card 19) forces exactly one re-run of that batch through the existing failed-batch recovery path (D6's recovery half).
- Forcing the failure: when record-batch records a fork's report, or `PersistRecoveryTerminal` records a recovery's terminal digest, and the batch holds an `AmendedCards` entry with `Rendered: false`, the batch first runs every normal check and is then recorded failed whatever the report's status, through `failBatch`.
  `failBatch` itself adds, for every unrendered entry, a `card_amended` reason naming the card ("card NN-slug was amended after this attempt began; recovery re-runs the batch on the amended card"), so a batch that another check already failed carries it too, and it never enters `Uncheckable`.
  `BatchFailedError` gains `CardAmended`, true when such a reason was added.
- The envelope: record-batch's and recover-batch's `batch_failed` envelopes carry `card_amended: true` when the error's `CardAmended` is set.
- Recovery: `card_amended` is recoverable, so `RecoverSpawnOrAttach` never answers `needs_fresh` for it.
  Every recovery strand `recoverSpawn` spawns while the batch holds any `AmendedCards` entry gets, in its failure digest (`failureDigestBlock`), one instruction per amended card to re-read the card and bring the committed work in line with it; the block is rendered from `AmendedCards`, not from the previous failure's reasons, so a stuck, dead or failed recovery does not erase it.
  The spawn carries the entries onto the fresh record with every `Rendered` set true, so that recovery's own report is not forced failed.
- Clearing: a recovery whose report is recorded done clears the batch's `AmendedCards`; one that ends stuck, dead or failed keeps them, so the next `recover-batch` renders them again.
Master's template gains a rung: `recover-batch <NN>` refusing with `{"batch_failed": true, "card_amended": true}` runs `recover-batch <NN>` again, backgrounded, even though the failed attempt was a recovery; every other recovery failure keeps the existing stop.
Bound: each amendment forces at most one failed record and so at most one extra recovery from Master's ladder, because only an unrendered entry forces the failure and the first spawn that renders it sets it true; a recovery that records done is trusted, under the same checks as any card, to have followed the amendment.
`TestFailBatch` in `batchfail_test.go` gains rows pinning the added reason and flag; `TestRecordBatch_AuditOutcomes` in `recordbatch_test.go` gains a row recording a done report over an unrendered amendment as failed with `card_amended`; `recoverbatch_test.go` gains `TestRecoverBatch_AmendedCards`, one sequence test (`test-budget.yaml` raises `internal/websterengine` by one) covering rendering, the `Rendered` flip, no forced failure on the rendering recovery's own done record, the re-edit forcing it failed, a stuck recovery keeping the entries for the next spawn, and done clearing them; `TestRecordBatchCmd_FailedBatchEnvelope` and `TestRecoverBatchCmd_RunningThenTerminal` in `verbs_test.go` gain steps asserting `card_amended` in each envelope; `claims_test.go` pins the rung.
`refusal-spec.md` gains the card-amended row, and the package doc describes the amendment's path.

**ImpactSummary:** Adds a forced-failure branch to record-batch and the recovery persist, an envelope flag on both verbs and a Master ladder rung; batches with no amendment behave as before.
