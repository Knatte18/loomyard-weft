# Card 38 — run-exit-pending-findings

**Edit:**
- `internal/websterengine/state.go`
- `internal/websterengine/auditledger.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/auditledger_test.go`
- `internal/websterengine/runlevel_test.go`

**Intent:**
Fix PR #321 finding 5: a run-exit correctness halt clears on a bare re-step.
`runExitAuditCrossCheck` deliberately does not record a correctness finding, and finding identity is keyed on the Master session, so `lyx webster run` without reverting the named paths starts a new Master that re-audits only its own transcript and reports done.

The finding offers two fixes: re-check the suspect paths, or require an explicit acknowledgement.
This card takes the acknowledgement, because a content re-check cannot clear every correctness finding soundly:
a suspect `_lyx/webster/state.json` changes on every save webster itself makes, a fork-contract write names a contract file the next Master rewrites, and a mutating fabric reference (card 36) names a command, not a path.

- `State` gains `PendingAuditFindings []PendingAuditFinding` (`json:"pendingAuditFindings,omitempty"`).
  `PendingAuditFinding` carries `ID` (the finding identity), `Class`, `Detail` and `Paths []string` (the suspect paths, empty for a pathless finding).
- `runExitAuditCrossCheck` appends each undispositioned correctness finding to `PendingAuditFindings`, once per identity, and saves state under the lease it already holds, before returning the stuck reason.
  The stuck reason's way forward becomes: `revert or re-derive the named paths in the task worktree with git, then run "lyx webster accept-audit" and re-step the Webster row (lyx webster run)`.
- `auditledger.go` gains `AcceptPendingAudit(st *State) []PendingAuditFinding`: it records every pending finding's identity in `AuditDispositions` with a new disposition value `accepted`, clears `PendingAuditFindings`, and returns what it accepted.
  It never saves; the caller holds the lease and saves.
- `Run`'s entry, after the fingerprint check and before Master spawns, refuses while `PendingAuditFindings` is non-empty, with a new exported sentinel `ErrPendingAuditFindings`:
  `webster: %d correctness finding(s) from an earlier run exit are pending: <details>; suspect paths: <paths>; way forward: revert or re-derive the named paths in the task worktree with git, then run "lyx webster accept-audit"`.
  `--fresh` builds fresh state and so drops them, as it drops every other record.
- `doc.go`'s run-exit paragraph states that a correctness finding stays pending until `accept-audit`, and that run entry refuses meanwhile.

Tests:

- `auditledger_test.go`: `AcceptPendingAudit` records each identity as `accepted`, empties the list, and a second call accepts nothing.
- `runlevel_test.go`: update the existing Master-`parent-write`-after-the-last-batch scenario: the run ends `stuck` naming the path and `lyx webster accept-audit`, and `state.json` carries one pending finding with that path; a second `Run` without accepting refuses with `ErrPendingAuditFindings` naming the path and `accept-audit`, and spawns no Master; after the test reverts the path with `git checkout` and calls `AcceptPendingAudit` and `SaveState`, the next `Run` ends `done` with no pending finding left.

**ImpactSummary:** Run-exit correctness findings persist as pending in state.json and block run entry until accepted; run-exit policy warnings are unchanged.
