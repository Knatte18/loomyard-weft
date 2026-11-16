```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked the sentences the fix added or rewrote against the code at HEAD.

1. **MEDIUM**: D1 now reads "repairing it or restoring its last committed version from the weft's history; a corrupt state file never counts as missing".
   - **Problem:** For a corrupt Webster state file in the middle of a run, there may be no committed version to restore.
   - **Evidence:** `websterengine.Dir` puts Webster's state under `_lym/webster` (`internal/websterengine/state.go:30-35`), so a commit would land in the weft. But the support log's Turn 4 says the implementer spec commits the run directory "only at Done", and D1's spec edit changes that only to *allow* commits during a run. D9's commit after every batch covers Weaver only. In the code, `CommitRecordsPaths` is called only from `internal/fabricengine/{origin,add,remove}.go`, and Webster has no such call. Removing the move-aside way forward leaves a corrupt Webster state file in a run that hasn't finished with no reachable way forward except hand repair.

2. **MEDIUM**: D6 now reads "whatever ended it: … or a fault in Weaver's own config or state. State first records the attempt's outcome and failure cause (D9), and then the strand is stopped".
   - **Problem:** The stop now depends on a state write, and a fault in Weaver's state is one of the listed causes. The fix doesn't say what happens when that write is the thing that fails.
   - **Evidence:** If the state can't be loaded or saved, the outcome can't be recorded. The strand then either isn't stopped, which breaks the fix's own guarantee, or is stopped with nothing recorded, which is the state F13 meant to close.

3. **LOW**: D12 now reads "so a strand still re-prompted on a failing gate is awaited until its gates pass".
   - **Problem:** It overstates the attached Wait. Wait evaluates the gate when the agent finishes even when every output file exists (`internal/shuttleengine/wait.go:36,50`), but the gated loop has a cap: "The gated loop then ends with the attempts spent so far" (`wait.go:171`).
   - **Evidence:** So the strand is awaited until its gates pass or its re-prompts run out, and the second case is a D6 failure. The D12 acceptance line "the batch is not recorded done until its gates pass" still holds.

Checked and holding:
- `AttachGated` reconstructs the run with the caller's gate (`attach.go:300-363`).
- `Geometry` has `WebsterDir`, `ReportsDir`, `PromptsDir` and `ScratchDir`.
- `mapPartition` is the mapping Webster's verbs use (`partition.go:77,90`).
- `RestampPlanBaseline` exists (`fingerprint.go:255`).
- `RunInFlight`'s rule matches the D13 wording.

Tool calls:
- Grepped `attach.go` for gate handling and looked up `websterengine.Dir`: my context lacked how `AttachGated` carries the gate and where Webster's state lives.
- Read `state.go:30-40` and grepped `wait.go` for gate handling: my context lacked whether a gated Wait checks gates when every output file exists.
- Grepped for `CommitRecordsPaths` callers and `mapPartition`: my context lacked whether Webster commits its state while a run is going, and the partition-mapping callers.
