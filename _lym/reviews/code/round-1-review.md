---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/boardcli/doc.go:114
    summary: doc.go line 114 says the refusal names the operator, the hub orch or the beam's orch, but a beam orch outside its beam now gets the parent orch
    fix: In internal/boardcli/doc.go reword line 114 so it says the refusal of a session that is no orch of the entry's beam names the operator, the hub orch or the orch of the entry's beam, and leave the added sentences at lines 118-120 as the beam-orch case.
    origin: lens:code-affected-set
---

### [LOW] doc.go line 114 does not mark the beam-orch refusal as the exception

**Location:** internal/boardcli/doc.go:114
**Issue:** Line 114 states the refusal "names the operator, the hub orch or the orch of the entry's beam as the way forward", unqualified.
The code at `runsettings.go:99-102` still produces that text for a non-orch role, but a beam's orch outside its beam now gets `refuseBeamOrchOutsideBeam`, which names its parent orch (`runsettings.go:117`, `:127-136`).
The added sentences at lines 118-120 say so, but a reader of line 114 alone is told the old, now partial, rule.

## Rejected

- Placement of the new orch-role paragraph after the decision-recording bullets in `## The board`: it sits in the section the record's Wanted names as likeliest, the record allows `## Messaging` or `## Working rules` only as "most likely", and the content matches the amended entry's Decision (parent beam not prime, wish not order, parent decides placement, pass upward).
- `ParentBeamOf` error replacing the original refusal: reachable only after `BeamUnder` read the same store without error, so the write is refused either way and no state is left inconsistent.
- No test row for the `ParentBeamOf` error branch: an internal pass-through of a board-load failure, no trust boundary.

Focus verdicts: `ParentBeamOf` returns "" for a prime parent and for a slug with no entry, so the named target is `ly:orch`, a valid creel target.
Every branch of `refuseRunSettingsCaller` where a beam's orch is refused outside its beam returns the new refusal, and a non-orch role keeps the old text.
The orch-role creel command and strand-name form match `lym creel post` and `agentname.Name.String`.
The new test rows pin the parent-orch target and the `inner` entry disturbs no other row.
The diff changes no refusal or board-write boundary beyond the way-forward text.
