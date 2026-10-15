---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:47
    summary: D2 puts the Run-entry removal before anything else without placing it against run.lock, and states no behavior for a failed removal at either removal point, though only a failed write is specified
    fix: In D2, replace "removes the record first thing at entry, before the auto-rebaseline" with "removes the record once it holds run.lock and before the auto-rebaseline", so a Run refused with ErrRunBusy never deletes the record of the live Run, and replace "before anything else at Run entry" in the Rationale likewise. Add one sentence stating that a removal failure, at Run entry or at begin-batch, is logged at Warn and does not change the refusal or the Run outcome, and add to Open risks that a record surviving a failed removal can route a later stuck outcome of the same Webster call, bounded by the next successful removal or begin-batch.
---

### [LOW] D2 leaves the removal's place against the run lock and its failure behavior open

**Location:** `_lyx/discussion/decision-record.md:47-52` (D2), against `internal/websterengine/runlevel.go:345-380`.
**Issue:** D2 says `websterengine.Run` removes the record "first thing at entry" and "before anything else at Run entry".
`Run` makes the scratch dir and then tries `run.lock`; a second `Run` finding the lock held returns `ErrRunBusy` there.
A removal placed literally first runs in that refused `Run` too, so a `lyx webster run` or a second loom step started while a Webster call is live deletes the live call's record.
The live call's adapter then finds no record and halts plain where it should have rerouted.
This is the same record-lifecycle guard the decision builds, with its bound unstated at this one point.
D2 also fixes what a failed record write does (logged at Warn, refusal unchanged) but not what a failed removal does.
A removal that fails and is ignored leaves exactly the stale record the lifecycle is meant to exclude, and one that fails the verb or the Run adds a refusal that has no spec row.
The plan writer has to pick, and `PATTERN-refusal-way-forward` makes a new refusal cost a row.
The loom adapter's error arms never read the record, so the exposure is limited to a stuck outcome later in the same Webster call, which is why this is LOW.

## Rejected

- Reroute-cap halt at the Webster row (focus 1): not a defect. D3 states the way forward for a plan conflict (edit the named pending cards by hand, then `lyx loom resume`), supplies it as loom's told `Shed.RerouteCapWayForward` made row-neutral, and notes the cap reason also quotes the adapter's reroute reason. Against the code, `reroute.go` ends the cap reason with the told clause after `way forward:` and quotes `output.Reason`, so D3's description holds. D5 adds the loom refusal-spec row for the capped halt, and the acceptance criteria pin it, so `PATTERN-refusal-way-forward` is met.
- Last-routed timing against the cap (focus 2): not a defect. D3, its Bound, the Open risks and a rejected alternative in the support log all state the chosen behavior: the file is written at verdict time, a capped conflict counts as routed, and resuming without the hand edit halts as a repeated conflict. The code agrees: `run.go` sends a Stuck with `Reroute` to `rerouteStuck`, which blocks at the cap with `RerouteCapped`, resets the row's count, and leaves `Run` to remove the record on re-entry while the last-routed file stays. The repeated-conflict halt can therefore fire for a conflict no plan review saw, and the record says so.
- Conflict-record lifecycle across begin-batch (focus 3): not a defect. D2 puts the removal as the verb's first step, before batch-number parsing, plan parsing, lock acquisition, state load and batch derivation, so none of those refusals can leave or see an earlier record, and its claim that a record exists only after the latest begin-batch refused with `plan_conflict` matches. The acceptance criteria pin the non-`plan_conflict` paths. The wording of the claim is also consistent with D3's remaining exposure, which covers a stuck for another reason after a `plan_conflict` refusal.
- Markdown structure of D3 (focus 4): not a defect. The three-item bullet list at lines 61-63 is closed by a blank line at line 64, so the paragraph at line 65, Bound and Rationale render as separate blocks.
