# Card 7 — delete-blocked-by-later-card

**Edit:**
- `internal/websterengine/batchfail.go`
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`
- `contracts/stencils/webster/webster-template-master.md`
- `internal/websterengine/batchfail_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/webstercli/verbs_test.go`

**Uses:**
- `plan:internal/planglyph#LaterDeleteReferences`

**Intent:**
Implement decision D4 (#401's second half): a `delete-not-done` failure caused by a later card still referencing the target names that card and a way forward that does not loop through `recover-batch`.

Unbegun cards are the cards of every batch with no record in state.json, the scope card 1 and card 2 use at dispatch; one helper in `recordbatch.go` computes them for both verbs.

record-batch: when the post-batch done-checks fail and at least one finding is `delete-not-done`, run `planglyph.LaterDeleteReferences(plan, <the batch's own cards>, <unbegun cards>, worktreeRoot)`.
When it reports a reference, the batch still fails through `failBatch`, and the batch-failed reasons additionally name the later card and the reference (file and line); the `BatchFailedError`'s way forward becomes: move the delete to a card after that one, run `lyx webster rebaseline --card NN` naming the edited cards, then `lyx webster recover-batch NN`.
When the failing batch's record also lists `Uncheckable` entries, the way forward is instead the `freshRestartSteps` that `recover-batch`'s uncheckable refusal names, since `recover-batch` would refuse that batch.
Every other batch failure keeps `lyx webster recover-batch NN` as its way forward.
Give `BatchFailedError` the field that carries the alternative way forward (its `Error()` keeps the default when the field is empty), filled through `failBatchInput`.
The same rule applies where `PersistRecoveryTerminal` fails a recovered batch on the shared post-batch pass, since both verbs run `postBatchChecks`.

recover-batch: in `RecoverSpawnOrAttach`, after the existing refusal of a failed batch carrying uncheckable paths (which keeps its own `run --fresh` way forward) and before `recoverSpawn`, run the same check over the batch's own cards against the unbegun cards.
While it reports a reference, refuse before spawning, with the reference named and the same plan-edit way forward, so `recover-batch` never names itself as the way forward from a state it cannot change.
An attach to a live, non-terminal recovery record is not affected.
A quarry infrastructure error from the check refuses as the check's other callers do (wrapped `ErrQuarryUnavailable`, re-run the verb).

Bound, stated in `RecoverSpawnOrAttach`'s doc comment: recover-batch refuses before spawning only while the check fires for the batch's own cards against the current plan and tree; moving the delete and rebaselining clears it.

Docs: refusal-spec rows "card not done: delete still referenced by a later card" (record-batch, and recover-batch's terminal persistence) and "recover-batch: delete still referenced by a later card", each with the plan-edit way forward and the `--fresh` variant for an uncheckable record; `doc.go`'s record-batch and recover-batch paragraphs.
In `webster-template-master.md`'s failure ladder, where a failed batch leads to `recover-batch`, say Master follows the way forward the failure names, and that a plan-edit way forward is not Master's to take: it ends the run stuck naming it.

Tests:
`batchfail_test.go` (`TestBatchFailedError_TextAndSentinel`): the default way forward is unchanged and an alternative one replaces it.
`recordbatch_test.go`: a batch whose Delete card leaves the target in place while an unbegun later card's Edit file still references it fails with reasons naming the later card and file:line and the plan-edit way forward; the same with an `Uncheckable` record names the `--fresh` steps; a `delete-not-done` with no later reference keeps `recover-batch`.
`recoverbatch_test.go`: `RecoverSpawnOrAttach` over that failed batch refuses before starting a strand (the fake starter records no start) with the same way forward; once the later card no longer references the target it spawns.
`internal/webstercli/verbs_test.go`: drive `record-batch NN` through its cobra command into this failure and assert the envelope names the later card and the plan-edit way forward, then `recover-batch NN` through its command refuses with exit 1 and the same way forward.

**ImpactSummary:** record-batch, recover-batch and recovery persistence gain a later-reference check on delete-not-done that swaps the batch-failed way forward and gates the recovery spawn; other failures are unchanged.
