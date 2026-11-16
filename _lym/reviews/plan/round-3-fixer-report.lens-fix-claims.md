```yaml
lens: fix-claims
outcome: reported
```

Task: check that each claim the fix added or rewrote holds against the code at HEAD.

**LOW**: the rewritten `batchGateCalls` bullet in card 9 (`09-batch-attempt-strand.md`) relies on a main-packages helper that doesn't exist for the call form it uses.
- **Changed text:** "one `card` call per card of the batch … (`GateCall("card", <card id>)`); then one `batch` call over every card of the batch, with the module's main packages when `last` is set."
- **Summary:** the bullet uses the `--wait` `GateCall` form for the batch's own calls. Card 13's `checkStoppedBatch` now runs those calls through `RunDeps.Verify`, which is a Go verify site. But `gateslot.Config` has no `--wait` variant that adds the main packages, so the `last` case names no existing call.
- **Evidence:** `internal/gateslot/tiers.go:378` defines `GateCall`, which renders the `--wait` form. The only main-packages variant is `AgentGateCallWithMainPackages` at `internal/gateslot/tiers.go:397`, which builds the call with `renderGateCall(key, false, cards)` and so has no `--wait`. A `--wait` main-packages call needs either `" --" + gateslot.MainPackagesFlag` appended to `GateCall`'s result, or a new gateslot method. Neither the card's Uses nor its Edit names one.

These claims hold:
- **`websterengine.Run` sets its stale files aside at entry:** `archiveStaleOutcome` and `ArchiveStaleSummary` are called at `internal/websterengine/runlevel.go:696-699`, inside `func Run` (line 377).
- **`outcomeRecord` has Webster's yaml keys:** `internal/websterengine/outcome.go:31-36` uses the keys `outcome`, `stuck_reason` and `batches_done`.
- **An unknown key is refused:** Webster's decoder sets `KnownFields(true)` at `outcome.go:49`.
- **`runplan.IsLastBatch` over the mapped partition:** it takes `[]batcher.Batch` (`internal/runplan/cardgate.go:74`), and that is the type `MapPartition` returns (`partition.go:23`).

Tool calls:
- Read `runlevel.go:690-702`, Webster's `outcome` struct, and `gateslot`'s `GateCall` and its variants, to check the entry-archive, key-set and gate-call claims, which my context showed only as grep lines.
- Found the function that encloses `runlevel.go:696`, and whether the archive functions tolerate a missing file, because the earlier grep didn't show the enclosing function.
