---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/discussion/decision-record.md, D8 and D14
    summary: D8 removes the round gate kind on the premise that it has no site left, but Webster's verify-gate fixer fork runs the hub's round gate call, so the removal reaches into Webster's own gate, which the record's Out list and the board entry verify-fix-escalation hold
    fix: In D8, drop the sentence that removes the round gate kind and keep round in the gate schema for Webster's verify-gate fixer fork (internal/websterengine/runlevel.go:664, internal/websterengine/doc.go:529), so that only the publish kind goes; in D14 and the matching acceptance criterion, prune only gate.yaml's publish list and keep its round list.
---

### [BLOCKING] Removing the `round` gate kind changes Webster's own gate

**Location:** `_lyx/discussion/decision-record.md`, D8 ("the `round` and `publish` gate kinds have no site and go from the gate schema"), D14 and the acceptance criterion that prunes "the `round` and `publish` gate lists".

**Issue:** D8 moves Darn-Write, Darn-Burler-Fix and Code-Burler-Fix to the `landing` call and concludes that nothing still uses `round`.
That premise is false: Webster's verify-gate fixer fork renders `gateConfig.GateCall("round")` at run entry (`internal/websterengine/runlevel.go:664`) and runs it before it reports (`internal/websterengine/doc.go:529`, "runs the hub's `round` gate call over the computed scope").
Removing the kind from the gate schema and pruning `gate.yaml`'s `round` list would make every Webster run refuse at entry, or would force a change to Webster's fixer gate.
The record puts "Webster's own gate and its fixer" out of scope and leaves them to the board entry `verify-fix-escalation`.
The board entry for this task asks only that `landing` be the one gate after Webster and that the `publish` gate go; it does not ask for the `round` kind to be removed.

## Board fit

Apart from F1, the design covers the entry's goal and Done-when:
the loom and darn tails, Merge-In without the lock and with the aligning Opus fixer, the Code-Perch after the merge, Describe before the lock, the PR-Review stub behind separate loom and darn flags, Finalize's one locked block with capped route-backs that end in a stop to the orch, the removal of batten's verify retry, and the Gone list with its readers.
The open caps the entry left to Discussion are settled in D4 and D10.
Treating every Finalize conflict as a large merge is a stricter reading of "on a conflict … fix it", but it falls within the latitude the entry gives Discussion to define a large merge, and the support log records it.
The renames (`code_*`, `deploy_branches`) and D14's migration follow from the entry's "what reads them goes with them or is updated".
The record defers board tidy after landing to an orch rule and the `board-tidy-after-landing` entry, as the entry says.
`gate-runs-once` depends on this task and says it will map onto the new tail; D9 keeps the darn rewrite until `gate-runs-once` replaces it, so neither task duplicates the other.

## Overlaps

`lyx loom peers` lists no peer run and no parent commit (`own_row` 3), so no overlap exists and no landing order applies.
