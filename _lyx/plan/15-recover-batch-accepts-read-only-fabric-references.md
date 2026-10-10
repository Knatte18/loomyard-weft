# Card 15 — recover-batch-accepts-read-only-fabric-references

**Edit:**
- `internal/websterengine#AcceptBatchFabricReference` -> `func AcceptBatchFabricReference(st *State, geom Geometry, n int, readOnly func(cmd string) bool, rerun string) (accepted []string, err error)`
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
`AcceptBatchFabricReference` takes the verb its evidence refusal names for the re-run, `lyx webster accept-audit --batch NN` from the accept-audit verb and `lyx webster recover-batch NN` from recover-batch, and its bound sentence is rewritten: no command the classifier accepts writes tracked content, a ref, config or a remote, and `git status` may refresh the index's stat cache, which changes no content.
`RecoverSpawnOrAttach`, before its `needs_fresh` refusal, calls it with `deps.ReadOnly` when every `Uncheckable` entry is a pathless fabric reference whose recorded command the classifier accepts: on success it records the warnings on the batch and goes on to spawn the recovery; an evidence failure returns the `ErrAuditNotAcceptable` wrap unchanged, whose way forward names `recover-batch` for the re-run.
A command the classifier rejects still refuses `needs_fresh` with today's text.
`recoverBatchCmd` carries `audit_not_acceptable: true` on that refusal's envelope, and `acceptBatch` passes its own verb name.
The Master stencil's failure ladder gains the rung: `recover-batch <NN>` refusing with `audit_not_acceptable` writes `outcome: stuck` with a `stuck_reason` quoting the refusal, and stops, since Master cannot clean the tree; `TestMasterStencilCoversEverySingleFlagRefusal` in `cli_test.go` sees the new flag.
`doc.go`'s accept-audit passage and the refusal-spec rows `recover-batch: findings recovery cannot check` and `accept-audit --batch: not acceptable` carry the in-line route and the envelope flag.
`TestRecoverSpawnOrAttach` gains the acceptance rows: a failed batch whose uncheckable entries are all read-only fabric references spawns a recovery with the warnings recorded, a non-read-only entry still refuses `needs_fresh`, and a dirty tree refuses with the evidence text naming `recover-batch`; `TestAcceptBatchFabricReference_Refusals` pins the rerun verb in the text; `TestRecoverBatchCmd_NeedsFreshEnvelope` gains the `audit_not_acceptable` envelope row, and `TestAcceptAuditCmd_Refusals` pins the accept-audit verb's own rerun text.

**ImpactSummary:** One exported helper re-signed with its two callers (recover-batch's spawn phase and the accept-audit verb) and its test; one new envelope flag and one stencil rung.
