# Card 16 — recover-batch-accepts-read-only-fabric-references

**Edit:**
- `internal/websterengine#AcceptBatchFabricReference` -> `func AcceptBatchFabricReference(st *State, geom Geometry, n int, readOnly func(cmd string) bool, verb string) (accepted []string, err error)`
- `internal/websterengine#RecoverSpawnOrAttach`
- `internal/websterengine/acceptbatch_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/doc.go`
- `internal/webstercli#websterCLI.acceptBatch`
- `internal/webstercli#websterCLI.recoverBatchCmd`
- `internal/webstercli/verbs_test.go`
- `internal/webstercli/cli_test.go`
- `internal/webstercli/acceptaudit_test.go`
- `//contracts/stencils/webster/webster-template-master.md`
- `//contracts/specs/refusal-spec.md`

**Intent:** Let `recover-batch` accept, in line, a failed batch whose uncheckable findings are all pathless read-only fabric references, so Master's ladder needs no operator `accept-audit --batch` for them (D8).
`AcceptBatchFabricReference` takes the verb that accepts, `accept-audit --batch` from the accept-audit verb and `recover-batch` from recover-batch, and words both texts that name a verb from it: the evidence refusal's re-run, `lyx webster <verb> NN`, and each recorded warning's detail, `accepted by <verb>`, so a warning recorded through `recover-batch` names it in `summary.md`.
Its bound sentence is rewritten: no command the classifier accepts writes tracked content, a ref, config or a remote, and `git status` may refresh the index's stat cache, which changes no content.
`RecoverSpawnOrAttach`, before its `needs_fresh` refusal, calls it with `deps.ReadOnly` when every `Uncheckable` entry is a pathless fabric reference whose recorded command the classifier accepts: on success it records the warnings on the batch and goes on to spawn the recovery; an evidence failure returns the `ErrAuditNotAcceptable` wrap unchanged, whose way forward names `recover-batch` for the re-run.
A command the classifier rejects still refuses `needs_fresh` with today's text, and so does every entry when `deps.ReadOnly` is nil: a nil classifier accepts nothing, as card 15 states for `ClassifyViolation`, so the in-line route is skipped and `AcceptBatchFabricReference` is never called with a nil classifier.
Bound: the in-line route clears a correctness-class audit halt without an operator only for a failed batch whose every uncheckable entry is a pathless fabric reference the classifier accepts, which sees static shape only, and only under `AcceptBatchFabricReference`'s evidence rule, HEAD at the batch start, or the start an ancestor and the tree clean; anything else keeps today's refusal.
`recoverBatchCmd` carries `audit_not_acceptable: true` on that refusal's envelope, and `acceptBatch` passes its own verb name.
The Master stencil's failure ladder gains the rung: `recover-batch <NN>` refusing with `audit_not_acceptable` writes `outcome: stuck` with a `stuck_reason` quoting the refusal, and stops, since Master cannot clean the tree; `TestMasterStencilCoversEverySingleFlagRefusal` in `cli_test.go` sees the new flag.
`doc.go`'s accept-audit passage and the refusal-spec rows `recover-batch: findings recovery cannot check` and `accept-audit --batch: not acceptable` carry the in-line route and the envelope flag.
`TestRecoverSpawnOrAttach` gains the acceptance rows: a failed batch whose uncheckable entries are all read-only fabric references spawns a recovery with the warnings recorded, each warning's detail naming `recover-batch`; a non-read-only entry still refuses `needs_fresh`; a nil `ReadOnly` refuses `needs_fresh` over read-only entries; a mixed batch of read-only fabric references beside an uncheckable entry of another kind refuses `needs_fresh`; a read-only fabric reference that carries a path refuses `needs_fresh`; and a dirty tree refuses with the evidence text naming `recover-batch`.
`TestAcceptBatchFabricReference_Refusals` pins the verb in the refusal text and the recorded detail; `TestRecoverBatchCmd_NeedsFreshEnvelope` gains the `audit_not_acceptable` envelope row, and `TestAcceptAuditCmd_Refusals` pins the accept-audit verb's own rerun text.

**ImpactSummary:** One exported helper re-signed with its two callers (recover-batch's spawn phase and the accept-audit verb) and its test; one new envelope flag and one stencil rung.
