# Card 45 — accept-audit-sync-failure

**Edit:**
- `internal/webstercli/acceptaudit_test.go`

**Prosa:**
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/webstercli/acceptaudit.go`
- `internal/webstercli/sync.go`
- `internal/webstercli/beginbatch.go`

**Intent:**
Fix PR #321 round-2 finding B: `accept-audit` refuses on a fabric sync failure (`acceptaudit.go`, the `fabricSync` branch), but the refusal table's `fabric sync failed` row names only the bracket verbs, `run` and `rebaseline`, and no test reaches the refusal, which breaks the Refusal Way-Forward Invariant.
The message already ends in `fabricSyncWayForward`, so the verb's code does not change.

`refusal-spec.md`: the `fabric sync failed` row's trigger reads "a bracket verb, `run`, `rebaseline` or `accept-audit` cannot commit its state to the fabric"; its class and way forward stay as they are, since `accept-audit` carries the same clause.

`acceptaudit_test.go` gains `TestAcceptAuditCmd_FabricSyncFailureNamesWayForward`.
It builds the verbs fixture with one pending finding, as the accepts-pending test does, but clears the skip-git environment switch `fabricengine.EnvSyncOptions` reads (`t.Setenv` to the empty string, so an ambient value cannot skip the sync) and sets `fx.CLI.openFabric` to a closure returning a sentinel error.
It asserts:

- the verb exits non-zero;
- the error envelope carries `audit findings accepted but the fabric sync failed`, the sentinel's text, and `lyx fabric commit`, the way forward;
- the reloaded `state.json` has no pending finding and records the finding's identity as `accepted`, since the state is saved under the lease before the sync runs.

**ImpactSummary:** Test and refusal-table row only; `accept-audit`'s behaviour is unchanged.
