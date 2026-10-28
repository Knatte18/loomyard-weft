```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end against the code and checked whether its end state meets its stated purpose.

**Findings**

1. MEDIUM — D5 (docs). D5 misses one stale line in the orch role stencil, which D1 makes false.
   - Evidence: `contracts/stencils/orch/orch-template-role.md:73` says "the released run takes the lock again at its next lock-taking row and reroutes if the parent moved meanwhile".
   - Walking a lock release through D1: the released run re-enters Finalize, merges the moved parent and gates it with no reroute.
   - D5 rewrites only the `Landing halts` section and names nothing about line 73, so the orch is told something that no longer happens.

2. LOW — D2. "No review input, its nil-is-absent value" is not what removing darn's override actually produces.
   - Evidence: `internal/loomcli/wiring.go:735` first sets `ReviewInput: fixerReportInput(...)` for every run.
   - `darnwiring.go:239` only overrides it for darn.
   - If the override is just deleted, darn inherits loom's Parent-Align fixer-report seam. That seam answers not found, because darn has no Parent-Align, but the value is not nil.
   - `envkit.go:117` lists `ReviewInput` as a field the drift guard expects filled.
   - The record should say what darn's value becomes: inherit loom's seam, or set nil explicitly and check that the Bouncer accepts nil.

3. LOW — D4. The pawl case of the arbiter's resume path is not stated.
   - D4 says the arbiter records the deciding decision with `lym loom decision add`, then resumes.
   - A pawl run has no Discussion record, so that verb may be refused there, and the arbiter then hands on.
   - The record's purpose still holds because a hand-on reaches the orch. Only the claim that the arbiter "can likely settle" it fails for pawl, and the record does not say so.

**Decisions walked**

- **D1**, loom Finalize, case: the parent moved and conflicts in a file the task changed.
  - The merge goes through the resolver, and the conflict session runs under the lock.
  - `sessions.count()` goes up, so the merge counts as conflicted, and the merge record is written.
  - With the large-merge switch removed, Finalize goes on to the `finalize` gate. A passing gate leads on to the parent records, the lock re-check and the landing.
  - Purpose met.
- **D1**, retry case: the parent-side merge returns merge-in-required.
  - `mergeAndGate` runs again and now gates instead of rerouting, then lands.
  - Purpose met.
- **D2**, darn case: a large merge.
  - No note is written and Finalize goes on to the gate.
  - Purpose met, apart from finding 2.
- **D3**, Finalize case: the gate fails, the fixer reports `escalate`, `Spent` goes down by one, and the halt reason gets the new prefix.
  - `landingshed.HandsBack` is false, so the envelope gets no `hand_back` mark.
  - `arbiterstop.HandsBack` (landing or decision prefix) is false, and `Check` reads it as an eligible blocked stop.
  - Purpose met.
- **D4**, same stop:
  - In `decideDrive`, the `stop.HandBack`, budget and precondition rows do not match, and neither do the hand-over and resume-returned rows (no build change, no tackler attempt).
  - `facts` calls `JudgesFirst(producer, reason)`. `Stop` carries `Reason`, so that input exists. It returns true, so the arbiter row fires before the tackler row.
  - The arbiter's `resume` with goto Finalize passes the Finalize dirt guard.
  - Finalize takes the lock and the merge is up to date. The failed tree is not skipped, because only a tree recorded as passed is.
  - The gate fails again, and the fixer re-spawns on the same Baseline and attempt count with the added decision now in the record.
  - On a hand-on, the same stop key returns the earlier verdict with no new spawn, and the driver hands back to the orch.
  - Purpose met.
- **D4**, Parent-Align case: a resume to the same row keeps its row entry, and the attempt record keeps the fork point. So an up-to-date re-merge still re-runs the fixer.
  - Purpose met.
- **D5**: met, apart from finding 1.

**Tool calls**

- Read `sheddriver`'s `Stop` struct and `ArbiterFirst` uses, to confirm a stop carries a reason for `JudgesFirst`. My context had only `drive.go`'s `facts`.
- Read the `decideDrive` rule table, to check that the escalate stop's path through the rows reaches the arbiter row. My context had only its doc.
- Grepped `ReviewInput` across production, and found `shedarbiter.Seam`, to see what darn's value becomes once the overlap-note override goes. My context had only the darn wiring line.
