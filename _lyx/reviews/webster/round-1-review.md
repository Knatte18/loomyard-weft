---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/parentreview/gate.go:339
    summary: closeAlignment dereferences a nil Aligner when a round holds a running alignment record but the gate was rebuilt with alignment off
    fix: "In closeAlignment, call c.cfg.Aligner.Close only when c.cfg.Aligner != nil, and settle the record through Store.SettleAlignment either way. Add a TestGate_Alignment row that builds the gate with a nil Aligner over a round whose alignment.json is running and asserts a recorded parent verdict, the wait bound and final() each settle it as stopped without a panic."
  - id: F2
    severity: NIT
    class: consistency
    location: internal/loomengine/template.yaml:40
    summary: the parent_review_wait_min line lost one of the two spaces before the comment marker that every other template line carries
    fix: In internal/loomengine/template.yaml, restore two spaces between the value 60 and the # on the parent_review_wait_min line.
---

### [MEDIUM] Nil Aligner panics on a running alignment record

**Location:** `internal/parentreview/gate.go:339`, reached from `gate()` (verdict branch and wait-bound branch) and `final()`.

**Issue:** `Round.aligning()` reads only the stored `alignment.json` state.
`closeAlignment` then calls `c.cfg.Aligner.Close` unconditionally.
`newParentReviewAligner` returns a nil `Aligner` whenever `alignmentOff` is true, and the wiring is rebuilt on each driver start.
So an operator who sets `skip_alignment: true` (or removes the board dir) to get past a stuck agent, then restarts, gets a nil-pointer panic at the first evaluation that finds a running record: a parent verdict, the wait bound, or the writer's end.
That is the escape route the config key exists for, so the fallback to today's path fails exactly there.
The plan's criterion that `skip_alignment` leaves parent review as before is not met for a round that began while alignment was on.

### [NIT] Spacing of the reflowed `parent_review_wait_min` line

**Location:** `internal/loomengine/template.yaml` (line `parent_review_wait_min: 60 # minutes ...`)

**Issue:** The diff touched this unrelated line and dropped one of the two spaces before `#` that every other template line carries.

## Rejected

- The focus question on `Session.Begin`: it returns any attempt with the same key, closed or not, and `Poll` returns `CauseClosed` without reading a closed attempt's verdict file.
  `PRAlign.Call` returns `Done` before `Begin` for a closed attempt of the key, and `TestSession` covers both.
  Not a defect.
- The focus question on the shared approval write: `approveVerb` and `rejectVerb` now call `RecordApproval` and `RecordRejection`, as does `PRAlign`; only the removal-failure error text moved from "remove the pending ..." to "record the ...", which card 9 prescribes.
  Not a defect.
- The focus question on `ResolveAlignment` and `alignment_pr_approve: false`: `ResolveAlignment` returns settings only, as card 1 specifies, and `landingAlignmentDeps` carries `PRApprove` into `PRAlign`, which writes the note instead of an approval.
  Not a defect.
- The focus question on alignment record paths: records land under `shedrun.AlignmentDir`, and `statusCommitPathspec` stages `shedrun.AlignmentRel`, pinned by `wiring_commitstatus_test.go`.
  Not a defect.
- The focus question on test budget and docs: `lyx gate test` was green for the touched packages, `cmd/lyx` and `contracts/...`, `test-budget.yaml` was raised, `doc.go` and `docs/overview.md` landed with the module, and no card calls for a `PATTERN.md` edit.
  Not a defect.
