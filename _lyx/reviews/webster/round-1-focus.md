---
round: 1
focus:
  - "Regeneration failure handling: check that every verb named in card 2 and card 5 logs at Warn naming `lyx fabric reconcile` and never fails or rolls back, and that only `lyx ide spawn <prime>` returns the error."
  - "Containment: check the hub workspace file and `_launchers` writes in `fabricengine` go through an `os.Root` at the hub, and that the declared `PATTERN-fabric-write-containment` and `PATTERN-hub-containment` changes land with their declared text."
  - "Card 4 `Delete` groups: check `assert-no-callers` holds for the removed portal helpers and accessor, and no reference to a retired portal name remains in code, docs or `doc.go`."
  - "Card 3 and card 5 Reconcile behavior: check launchers-only restore skips local pairs, `launchers_restored` and `hub_details` are reported as planned, and the retired-artifact sweep cannot remove anything outside the portal links and the old workspace file."
  - "Test economy: check the lock-wait tests use `RegenerateHubWorkspaceWithinForTest` so none waits out the production 5-second wait, and that no new test duplicates an existing one."
---

Seed focus drawn from the plan's Card Index, shared decisions and declared PATTERN changes.
