---
round: 1
focus:
  - "internal/loomcli/wiring.go: the parent replaced the per-row `Row*`/`Review*` fields with `Segments`/`Setting` via `configreg`, and the task changed the `SetNotifier` call to `orchcli.NotifyPrime(location, loomProgram, line)`. Settle whether the merged `wire` still hands every seam the task's notice path needs, and whether `loomProgram` and the notifier sit correctly in the parent's reshaped wiring."
  - "internal/testkit/envkit/envkit.go `nilLegal`: the parent removed the `Row*` entries and added `Segments` and `Setting`, the task removed the three `NoticeLog.Now` entries. Settle whether the merged list still matches the merged dep structs, so no seam is skipped that should be wired and none is wired that the list still exempts."
  - "PATTERN.md and pattern/PATTERN-config-strictness.md: the parent added `configreg` to the strict callers and reworded entries (`PATTERN-durable-vs-ephemeral-state`, `PATTERN-creel-store` and neighbours). Settle whether the task's creel and notice code, and any `configreg`/`Load` use it added, follows the changed wording."
  - "cmd/lyx/testdata/test-budget.yaml: both sides edited package rows. Settle whether the merged rows still fit the merged test counts for `internal/loomcli` and `internal/testkit/envkit`, and whether a test either side added now duplicates one from the other."
  - "Notice comments the task rewrote to say `posted as creel mail`: settle whether parent code that touches the same notice or notifier path still says or does `queued`."
---

Seeded from the overlap list; each entry names a file both sides changed and the question the interaction raises.
