```yaml
lens: walkthrough
outcome: reported
```

**Walkthrough lens: findings**

1. **BLOCKING, D9 / D10, `beam` shape: the beam never starts.**
   - Case: Sketch `foo` with an approved shape `beam`. Raddle-File runs `lym beam create foo` from inside the live raddle run, whose batten run is also keyed `foo`.
   - `createSteps` (`internal/beamcli/create.go`) reads `RunStatus(foo)` and finds the raddle's batten run not done, so it sets `RunLive = true` and starts no pawl run.
   - It clears a done run only when that run is done and its pair is gone. Neither holds while Raddle-File is running.
   - End state: the entry carries the beam flag and its owner, but no pawl run exists. BeamUp never runs, so D11's hand-off is never sent.
   - D10 says the raddle run is cleared on `lym beam create`, which contradicts D9 calling that verb from inside the same run.
   - The design needs the beam to start after the raddle pair is torn down. Either the parent orch or dobby runs `lym beam create`, or a batten step runs it after teardown.

2. **MEDIUM, D9 `tasks` shape: the Sketch entry is pruned at once.**
   - Raddle-File marks the Sketch entry done. The raddle's own batten run then reaches its Land row, which runs `board.Prune(true)` for a run that landed no code (`internal/battencli/land.go`).
   - That removes the done entry right away.
   - The support log rejected removing the entry so that its record stays linked from the README. The mechanism removes it anyway, one row later.

3. **MEDIUM, D7, interactive mode: nothing triggers the cold-proposal commit and the waiting notice.**
   - A row's gate runs at the session's handoff, when the session finishes on its output files.
   - In interactive mode the session must stay open after the proposal is written.
   - If `proposal.yaml` is the output file, the session finishes at its first write and never waits for approval.
   - If the approval record is the output file, the gate never runs before approval. Then nothing commits the cold proposal or posts "waits for the operator" (D7), nothing sets the "created after the cold proposal" cutoff (D8), and D14 has no cold input.
   - The record names no event between "proposal written" and "approved" for these steps to hang on.

4. **MEDIUM, D8: the re-check misses entries that landed or started running.**
   - `stampDates` (`internal/boardengine/dates.go`) leaves `Revised` unchanged on a status change. Landing to done and starting a run are both status changes.
   - So a `board_checked` peer in the same packages that landed or started between proposal and filing passes the "same `revised` stamp" test.
   - That is the board movement the entry asks to be caught: code under the parts changed.
   - The re-check should also compare status, at least the open, running and done states.

5. **LOW, D9 `tapestry` shape and D10: dobby can try to start the task while the raddle run is still live.**
   - The same write clears the run status and releases the re-typed task. The raddle's batten run still has its Land and Teardown rows to go.
   - Dobby sees a released task with an empty status and may try to start `foo` while the batten run for `foo` is live.
   - D10 covers only a done run with its pair gone. The record does not say whether dobby retries after that refusal or marks the task failed.

6. **LOW, D11: "a later start sends none" has no mechanism.**
   - The condition "the beam's entry has a stored raddle proposal" stays true for every start.
   - No sent-marker is named, so the acceptance criterion "a later start sends none" has no mechanism behind it.

**Decisions walked**
- D1: a task with recipe `raddle` runs Record-Write, Raddle-Write, Raddle-File. Purpose met.
- D2: the Sketch is promoted with recipe `raddle`. `sketchOf` keeps `discussed`; the operator releases a draft. Purpose met, assuming promote accepts a recipe; not checked.
- D3: Record-Write on a discussed entry, for darn and for raddle, copies the stored record. Purpose met.
- D4 and D5: no mechanism to break.
- D6: a proposal whose part carries a criterion naming another part's id passes. Fine, since the discussion check reads only decision→acceptance. Purpose met.
- D7: broken in interactive mode (finding 3). Met in auto mode.
- D8: misses landed or started peers (finding 4).
- D9: `tapestry` partly met (finding 5); `tasks` breaks the record link (finding 2); `project` met with D10; `beam` broken (finding 1).
- D10: `MakeSketch` is skipped by row. Met for start; broken for `lym beam create` (finding 1).
- D11: never reached because of finding 1; repeat-send gap (finding 6).
- D12 to D14: the usage file, then the cost report, then the raddle report. Met, except that D14's cold-proposal input depends on finding 3.
- D15: no mechanism to break.

**Tool calls**
- Read `internal/battencli/land.go` lines 120–175: to see what Land does for a run that landed no code; my context did not hold it.
- Grepped `Revised` in `internal/boardengine`: to learn whether status writes bump the stamp; not in context.
- Grepped `internal/beamcli` for its handling of an existing run: needed to see what beam create does when the slug's run is live; not in context.
- Read `stampDates` in `internal/boardengine/dates.go` and `createSteps` in `internal/beamcli/create.go`: to confirm when the revised stamp changes and the beam-create run check; not in context.
