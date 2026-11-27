```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, checking that the fix covers every member of each set-premise finding.

Findings left unchecked: none.

1. **LOW**: `_lym/landing/summary.md`, `## Changes`, first bullet (a sibling of the rewritten Design decisions bullet).
   - Summary: the bullet is stale after F2.
   - Evidence: it still says "`State.MasterNotePath` (`masterNotePath`) becomes `State.MasterNotePaths` (`masterNotePaths`)". `State.MasterNotePath`, tagged `json:"masterNotePath,omitempty"`, now exists again (`internal/websterengine/state.go:102-104`) as the read-back field for old states. The Design decisions bullet says the old key is read back, but this bullet reads as if the field were gone.

The other sets are covered:
- **F3/F6 (wording that allows only one note):** `audit.go`, `doc.go` and the `runlevel.go:722` comment now agree. The singular "friction note" lines left in `webster-template-master.md:308,319` and `contracts/orchrole/orch-role.md:527` describe what one Master is told to write, which the review's Rejected section accepts for the stencil.
- **F2 (readers of the state):** every non-test read of webster's `state.json` goes through `LoadState`, including `weaverengine/active.go:84`. Both `CheckParent` callers pass the list.
- **F5 (append guards):** both guards have a row.

Outside my lens: if an older binary writes `masterNotePath` into a state that already holds `masterNotePaths`, `LoadState` prepends that later path, so "in spawn order" (`state.go:97`) does not hold in that mixed-binary case. I found nothing that depends on the order.

Tool calls (2):
- `grep` for single-note wording across `internal`, `cmd`, `contracts`, `specs` and `pattern`: my context held only searches under `internal` and `docs`.
- `grep` for other readers of webster state and `MasterNotePaths` sites: my context had not covered callers outside `websterengine`.
