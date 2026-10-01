# Card 46 — pending-findings-block-run

**Edit:**
- `internal/shedadapters/webster.go`
- `internal/shedadapters/doc.go`
- `internal/shedadapters/webster_test.go`

**Prosa:**
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/websterengine/runlevel.go`
- `internal/shedengine/run.go`
- `internal/state/state.go`

**Intent:**
Fix PR #321 round-2 finding C: `ErrPendingAuditFindings` surfaces from the Webster row as a producer error, so the run goes `failed` and ly-drive treats it as a repair candidate.
It is a correctness halt only the operator can clear, by checking the named paths and running `lyx webster accept-audit`, so the run must go `blocked`, which ly-drive hands straight back.

`webster.go`, `WebsterProducer.Call`: beside the `ErrMasterAsking` branch, an error matching `websterengine.ErrPendingAuditFindings` through `errors.Is` maps to `shedengine.Stuck` with an empty Path, after the same `cancelErr` check the asking branch makes.
Its Reason is the error's own text with `, then re-step the <row> row` appended, `<row>` being the producer's name, so the reason keeps the suspect paths, `lyx webster accept-audit`, and a trailing way-forward clause.
The Webster row has no `on_stuck`, so shedengine persists `blocked` with that reason.
Every other run error keeps its current mapping.

`doc.go`'s WebsterProducer bullet adds `websterengine.ErrPendingAuditFindings` to the errors that map to Stuck, with the run-entry refusal's text as the reason.

Tests in `webster_test.go`:

- `TestWebsterProducer_PendingAuditFindingsIsStuck`: a fake runner returning a wrapped `ErrPendingAuditFindings` naming `internal/x.go` → `Call` returns Stuck, a nil error and an empty Path, and the Reason contains `internal/x.go`, `lyx webster accept-audit` and `re-step the loom row`.
- `TestWebsterProducer_PendingAuditFindingsBlocksRun`: a one-row `shedengine.Shed` (row `Webster`, no `on_stuck`) over that producer, its status file written through `state.UpdateJSON` at `Webster` in state `running` in a temp dir → `Step` returns no error and a result in state `blocked` whose Reason names `lyx webster accept-audit`; re-reading the status file shows state `blocked`, current producer `Webster`, and the same reason.
- `TestWebsterProducer_OtherEngineErrors` keeps every row it has, so the other run errors still surface as errors.

`refusal-spec.md`: the `pending audit findings` row's way forward quotes the reason a shed-driven run blocks with: revert or re-derive the named paths in the task worktree with git, then run `lyx webster accept-audit`, then re-step the Webster row.

**ImpactSummary:** A Webster row whose run entry refuses on pending audit findings now blocks the shed run with the accept-audit way forward instead of failing it; every other Webster error mapping is unchanged.
