```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I checked each pair of the record's decisions that act on the same state for a break.

**Pairs checked**
- D1 ↔ D2, the builder: D1 asks for beside-runs, and the builder reads the recipe from the seed of the orch's own location (`wireCLI` sets it to the beam). This means the same location as D1's status check. Held.
- D1 ↔ D2, the sidecar: D1 writes the status sidecar at the path D2 declares, and `orchcli` already imports `loomengine`, which owns the scratch directory. On the next `lym loom start`, `resolveStatusStrandAction` finds the guid with the current build and keeps it. Held.
- D1 re-add ↔ the bootstrap's add, both through `AddStrandUnless` on `loom-status`: whichever add wins, the other skips and writes no sidecar. Held.
- D1 replace ↔ the bootstrap's replace path, a stale status strand read before D1's replace: the bootstrap's `ReplaceStrand(staleGUID)` fails because the guid is gone, and it returns nil. D1's re-add supplies the strand. Both still hold, but the bootstrap logs a false warning (finding 1 below).
- D1 replace ↔ the bootstrap's replace landing first: D1's `ReplaceStrand` on the guid it read fails. Its warning names `lym loom start` and `lym beam start`, and a rerun of `lym beam start` reaches D1 again. This race predates the task. Held.
- D1 ↔ D3, window naming: D1's re-add carries group = recipe, size 1. `createOwnWindowPaneLocked` records `WindowGroups[group]` and names the window by the group when shared, otherwise by the role. D3 reads that same record and the strand's persisted `Display.Window`. Held.
- D1 ↔ D3, the done gate: both test `shedengine.StateDone`, so a done run gets neither a re-add nor a line. Held.
- D1 failure paths ↔ D3: with a failed re-add no status strand is tracked, and the D3 criterion "no window name" covers that. Held.
- D2 builder (no readable seed: no group, window `loom-status`) ↔ D3 (no readable seed: no line): consistent, each per its own text. Held.
- D3 ↔ the lock Constraint: `LoadState` takes the shared lock through `ReadJSON`. `shedrun.ReadSeed` imports no lock package, and the status cache polls without a lock. Held.
- D1's replace (viewer moves to the runs window) ↔ D1's re-add with `KeepViewer`: the re-add selects nothing, as the `newAgentWindowSelected` Constraint says. Held.
- Acceptance criteria ↔ D1, D2 and D3: every criterion matches the decision it names. The "role already held" criterion can only be reached through a fake seam, since D1 only re-adds after removing the strand; it is testable with the record's fake strand ops. Held.

**Findings**

1. LOW, design, `_lym/discussion/decision-record.md#D1`
   - Summary: D1's change routes only the bootstrap's add path through `AddStrandUnless`. Its replace path still loses a race with D1 and logs "the status pane will be missing this run" while D1's re-added strand is present. The run is not harmed; the logged message is wrong, and the record does not mention it.
   - Sequence: the bootstrap reads a stale status strand → D1 replaces it with the runs strand and re-adds a status strand → the bootstrap's `ReplaceStrand(staleGUID)` fails on the missing guid.
   - Evidence: `internal/loomcli/sharedbootstrap.go:302-309` (replace failure logs and returns nil), `internal/reedengine/replace.go:42-` (`removeStrandLocked` on an unknown guid errors).

No tool calls made.
