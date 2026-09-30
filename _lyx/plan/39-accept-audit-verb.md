# Card 39 — accept-audit-verb

**Edit:**
- `internal/webstercli/cli.go`
- `cmd/lyx/helptree_test.go`

**Create:**
- `internal/webstercli/acceptaudit.go`
- `internal/webstercli/acceptaudit_test.go`

**Uses:**
- `internal/webstercli/rebaseline.go`
- `internal/webstercli/sync.go`
- `internal/websterengine/auditledger.go`

**Intent:**
Expose card 38's acknowledgement as `lyx webster accept-audit`, the way forward the pending-findings refusal and the run-exit stuck reason name.

`acceptaudit.go` builds the verb on `rebaselineCmd`'s shape: `clihelp.ShouldAbort` first, the state-mutation lease, `LoadState`, `websterengine.AcceptPendingAudit`, `SaveState`, lease released, then `fabricSync` with message `accept-audit`, and a fabric-sync failure reported with `fabricSyncWayForward` as `rebaseline` reports it.

- No run in progress (no `state.json`) refuses as `rebaseline` does: `webster: no run in progress; run "lyx webster run" first`.
- Nothing pending succeeds with `accepted: []`, so a repeated call is harmless.
- The success envelope carries `accepted`, one entry per accepted finding with its `class`, `detail` and `paths`.
- `Short`: `accept the pending run-exit audit findings once their paths are checked`.
  `Long` says the operator runs it after reverting or re-deriving every suspect path the findings name, that it only records the acknowledgement and never changes the task worktree, and gives the example `lyx webster accept-audit`.
- `cli.go` registers the verb and adds it to the parent's verb list in its help text.
- `cmd/lyx/helptree_test.go`'s webster entry lists `accept-audit`.

`acceptaudit_test.go` (integration-tagged like the rebaseline verb tests, hub fixture from `internal/hubforge`): a state with one pending finding → the verb exits 0, the envelope lists it, and the reloaded state has no pending finding and records its identity as `accepted`; a second call returns an empty `accepted`; no `state.json` → the error envelope names `lyx webster run`.

**ImpactSummary:** Adds one webster verb and its help-tree entry; existing verbs are unchanged.
