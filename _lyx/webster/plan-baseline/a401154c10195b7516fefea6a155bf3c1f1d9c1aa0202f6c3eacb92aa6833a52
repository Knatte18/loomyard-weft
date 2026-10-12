# Card 2 — webster-blocking-delete-references

**Edit:**
- `internal/websterengine#laterDeleteReferenceReasons`
- `internal/websterengine_test#deleteReferencedBatches` -> `func deleteReferencedBatches(t *testing.T, worktree, target, user string) []batcher.Batch`
- `internal/websterengine_test#TestRecordBatch_DeleteNotDoneNamesLaterCard`
- `internal/websterengine_test#TestRecoverSpawnOrAttach`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:**
Webster's batch recording and `recover-batch`'s `ErrRecoveryDeleteReferenced` refusal count only blocking `delete-before-reference` findings (Decision 4).
`laterDeleteReferenceReasons` keeps only the findings of `Index.LaterDeleteReferences` whose severity is blocking, so an informational name-only method match is neither a reason in the failed batch's digest nor the trigger of `deleteReferencedWayForward`, and `RecoverSpawnOrAttach` neither refuses on it; a blocking package-level reference still adds its reason, picks the plan-edit way forward and refuses recovery, as today.
The function is the one site both callers reach, so `failCardNotDone` and `RecoverSpawnOrAttach` change with it.
Bound: a name-only method reference no longer fails a Webster batch; the plan gate already judged the typed answer before the run, and the later card's verify build catches a real call.

`deleteReferencedBatches` takes the deleted glyph and the body of `internal/foo/user.go`: `impl.go` declares `func Gone() {}`, `type Thing struct{}` and `func (Thing) Gone() {}`, card 1 deletes `target`, and card 2 edits `user.go`.
The existing rows pass `internal/foo#Gone` with the bare-call body (`Gone()` on line 4) or the body with no call.
New rows, each deleting `internal/foo#Thing.Gone` while `user.go` calls `t.Gone()` on line 4 of `func user(t Thing)`:
`TestRecordBatch_DeleteNotDoneNamesLaterCard` wants the batch failed with `way forward: lyx webster recover-batch 01` and without `2-later` or `rebaseline`;
`TestRecoverSpawnOrAttach` wants the failed batch to spawn its recovery.
The existing blocking rows stay and pin that a package-level reference still refuses.

`doc.go`'s delete-not-done paragraph and the two `refusal-spec.md` rows ("card not done: delete still referenced by a later card", "recover-batch: delete still referenced by a later card") say the check counts a reference resolved to the deleted member, and that a method matched by name alone does not refuse.

**ImpactSummary:** one filter in the shared reason builder reaches record-batch, persist-recovery-terminal and recover-batch; the test helper's two callers are the two tests listed.

**Commit:** 2: Webster counts only blocking delete-before-reference findings
