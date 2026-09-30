# Card 15 — batten-decision-resume

**Edit:**
- `internal/battenshed/deps.go`
- `internal/battenshed/innerrun.go`
- `internal/battenshed/innerrun_awaiting_test.go`
- `internal/battenshed/innerrun_test.go`
- `internal/battencli/wire.go`
- `internal/battencli/wire_test.go`
- `internal/battencli/run_test.go`
- `internal/battencli/records_integration_test.go`
- `internal/battenrecipe/fixture_test.go`
- `internal/shedrecipe/entries_batten.go`
- `internal/shedrecipe/entries_batten_test.go`
- `internal/shedrecipe/fixture_test.go`
- `internal/shedbuild/fixture_test.go`

**Uses:**
- `internal/landingshed/rejection.go`

**Intent:**
Make batten resume an awaiting child on a rejection exactly as on an approval (D8), so a batten-driven child awaiting at the gate does not sit forever after `reject`.
In `deps.go`, `ChildApproval` becomes `ChildDecision{Kind, At, HeadSHA string}`, with `Kind` one of two exported constants for `approve` and `reject`, and `InnerRunDeps.ReadApproval` becomes `ReadDecision func() (ChildDecision, bool, error)`, documented as reading whichever operator record the child's run holds.
In `innerrun.go`, `callAwaiting` reads the decision instead of the approval, and the acted-on identity becomes `<kind> <at> <head_sha>`, so an approval and a rejection at the same head within the same second never share one.
Rename the marker helpers and suffix to name a decision (`decisionActedFile`, `-decision-acted`) and reword every reason and `awaitingHandOff` to name both verbs;
an old `-approval-acted` marker is simply never read, which costs at most one extra resume of a still-awaiting child.
In `battencli/wire.go`, `ReadDecision` reads `LoomApprovalPath` and `LoomRejectionPath` in the task worktree through `landingshed.ReadApproval`/`ReadRejection`;
since `approve` and `reject` each remove the other's record first, at most one is present, and when both are the approval wins, matching the gate's own precedence.
In `shedrecipe/entries_batten.go`, the `requireSeam` call and the entry's doc comment name `InnerRun.ReadDecision`.
Tests: `innerrun_awaiting_test.go`'s approval cases run over `ChildDecision`, and new cases assert a rejection resumes the child once and not again for the same record, and an approval at the same head and second as an already-acted rejection resumes it again;
`records_integration_test.go` writes a rejection record and asserts one resume;
the fixture `InnerRunDeps` literals in `innerrun_test.go`, `run_test.go` and the battenrecipe, shedrecipe and shedbuild fixtures follow the rename, as do `entries_batten_test.go`'s nil-seam case and `battencli/wire_test.go`'s non-nil seam row.

**ImpactSummary:** Renames one exported battenshed type and one InnerRunDeps field; battencli's wiring and shedrecipe's batten entry seam check are their only production users, and every test building or checking InnerRunDeps follows the rename.
