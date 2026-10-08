# Card 16 — reset-sites-repoint

**Edit:**
- `internal/websterengine#reconcileHead`
- `internal/websterengine#reconcileReportHead` -> `func reconcileReportHead(git Git, worktree, reportHead, subject string, parentBranch ParentBranchFunc, batch int) (warning string, err error)`
- `internal/websterengine#headRefusal`
- `internal/websterengine#reportHeadRefusal`
- `internal/websterengine#parentMergeRefusal`
- `internal/websterengine#acceptAuditHeadRefusal`
- `internal/websterengine#AcceptBatchFabricReference`
- `internal/websterengine#RecoverSpawnOrAttach`
- `internal/websterengine#RecordBatch`
- `internal/websterengine#awaitTerminal`
- `internal/websterengine/doc.go`
- `internal/websterengine/gitwrap_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/acceptbatch_test.go`
- `internal/websterengine/suspect_test.go`
- `internal/websterengine/recoverbatch_test.go`
- `internal/webstercli#websterCLI.acceptAuditCmd`
- `internal/webstercli/acceptaudit_test.go`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/claims_test.go`
- `contracts/specs/refusal-spec.md`

**Delete:**
- `internal/websterengine#resetKeepStep`

**Uses:**
- `internal/websterengine#ResetToReportHead`
- `internal/websterengine#ResetToLastBatchHead`
- `internal/websterengine#ResetToBatchStart`

**Intent:** Every webster way forward that names `git reset --keep` names a `lyx webster reset` target instead (D2), so no agent or operator needs a raw git reset.
- `reconcileHead`'s refusals (a non-merge commit past the report head, and an unqualified merge) take their reset step from the `headRefusal` wording: for record-batch and recover-batch it is `lyx webster reset --to report-head --batch NN`, then the re-run, then the parent merge-in redo where it applies; `reportHeadRefusal` becomes a function of the batch number, and `reconcileReportHead` takes the number from its two callers (`RecordBatch`, `awaitTerminal`).
- `acceptAuditHeadRefusal`'s step is `lyx webster reset --to last-batch-head`.
- `AcceptBatchFabricReference`: the refusal for a start that is not an ancestor of HEAD names `lyx webster reset --to start`, then `lyx webster run`; the refusal for kept commits carrying a fabric reference that is not read-only (the start is an ancestor of HEAD) names `lyx webster reset --to batch-start --batch NN`, then re-running `lyx webster accept-audit --batch NN`.
- `RecoverSpawnOrAttach`'s needs-fresh way forward replaces its parenthetical `git reset --keep` with `lyx webster reset --to batch-start --batch NN`.
- `resetKeepStep` is deleted once no caller remains.
- accept-audit's long help names the lyx target where it names `git reset --keep`.
The target spellings come from the card-11 constants.
Master's template gains a rung: `record-batch <NN>` or `recover-batch <NN>` refusing with a head mismatch whose way forward names `lyx webster reset --to report-head --batch <NN>` → run that reset, then re-run the refused verb once; a refusal of the reset, a second mismatch, or a way forward that also needs a parent merge-in redone ends the run `stuck` quoting the refusal, since a merge-in is not Master's to redo.
Bound: the rung runs only the `report-head` target, which card 11 bounds, and at most once per batch.
Grep the tree for `git reset --keep` and `resetKeepStep`; every test asserting the old text joins this card (the files listed), each updated to assert the lyx target, and `claims_test.go` pins the new rung.
`refusal-spec.md` rewrites the "HEAD moved past the report", "unqualified merge after the report", "accept-audit: HEAD past last batch head", "accept-audit --batch: not acceptable" and "recover-batch: findings recovery cannot check" rows; the package doc drops its `git reset --keep` mentions.

**ImpactSummary:** Rewords six webster refusals and threads the batch number into reconcileReportHead's two callers; deletes resetKeepStep, whose callers are all in this card.
