```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case.

**Decisions walked**
- D1, a Sketch promoted with recipe `raddle` going through batten's rows Record-Write, Raddle-Write, Raddle-File: met, given D3.
- D2, a discussed note whose `sketch` label was dropped, promoted with `--recipe raddle`: met.
- D3, a raddle-filed darn part: Record-Write copies the part record, and Darn-Write reads it as its scope (`contracts/stencils/darn/darn-template-write.md:25`) without rewriting it. Met.
- D4, Raddle-Write's spawn reads the loom key `raddle`: met.
- D5 and D6, a part that builds D5 and carries the criterion "D5 and D6: …": met. The discussion check refuses only an id no bullet names, never an extra one.
- D7, interactive mode, proposal changed after approval: met. Raddle-File's hash match refuses it.
- D8, the agent reads the board at T0 and its first passing `raddle-propose` comes at T20. Entry Z, sharing a module label and missing from `board_checked`, is created at T10: **broken** (F2).
- D9, a `beam`-shape run that crashes after step 3, before step 4: **broken** (F1).
- D9, a re-check failure at step 3 followed by a changed, re-approved proposal: met. The return deletes the step 1 and 2 outputs, and their hash stamps force a rewrite.
- D9 and D8, the run's own entry's status change from Raddle-Write to Raddle-File after the baseline: met. Status writes do not bump `revised`; only a scope change does (`internal/boardengine/dates.go:48`).
- D10, `lym board raddle` by a filed slug after Land prunes the Sketch: met.
- D11, BeamUp of the pawl run that step 4 starts for the new project: met, unless step 4 never runs (F1).
- D12, a Claude session ending done with one fork on another model, priced per message: **broken** (F3).
- D13, a cell of motif and raddle runs with no size record: met, as a mean with no fit.
- D14, a fixture store with predicted cost against landed actuals: met.
- D15, the glossary row: met.

**Findings**

F1. BLOCKING, design, D9 (step 3 → steps 4 and 5) and its acceptance criterion "interrupted after any step and run again finishes the remaining steps".
- **Summary:** step 3 marks the run's own entry done and pushes the board. From then on origin reads the run as ended, so a halt or crash after step 3 cannot be resumed to finish the beam create and the notice.
- **Trace:**
  - The run ledger sets `Ended: isDone(task)` (`internal/boardengine/replay.go:272`).
  - Every batten resume runs `reconcileResume`. For an ended entry, `resumeRefusal` refuses with "origin ended … run lym cleanup --apply … instead of resuming it" (`internal/battencli/resumereconcile.go:114-123`).
  - The hub reconcile tears down an ended entry's pair (`internal/pairteardown/originseams.go:155`).
  - A weft replay sets aside the run's later local record commits. `withoutFinalizeEntered` spares only runs whose Finalize was entered, and raddle has no Finalize row (`internal/fabricengine/replay.go:307-320`).
- **End state:** the beam's project is filed with no beam and no notice. D9's Bound and Facts cover the run lock, `SetRunStatus`, `MakeSketch` and `Prune`, but not the origin-ended paths.

F2. MEDIUM, design, D8 (re-check baseline) together with D7 (`raddle-propose` runs only D6's checks).
- **Summary:** the baseline is "the latest passing check", meaning the `raddle-propose` or gate pass, not when the agent read the board. An entry created or revised while the agent works, before its first passing propose, is never compared.
- **Trace:**
  - The agent reads the board at T0 and fills `board_checked`.
  - Z, a task sharing a module label, is created at T10.
  - `raddle-propose` passes at T20 and sets the baseline to T20.
  - `raddle-approve` and Raddle-File test only entries revised after T20, so Z passes unseen.
  - The same holds in auto mode with the gate pass, and for each new propose after a return.
- **Evidence:** the decision record, D8's first line (line 104) and D7 (line 91).
- **Purpose missed:** "the board can move meanwhile, so the proposal is checked against the board again". The fix is to set the baseline at the agent's board read, for example a stamp recorded in the proposal and checked by D6.

F3. MEDIUM, design, D12 (usage writer prices "per message … through the engine's `MessagePricer`").
- **Summary:** the writer is handed the four session totals from `SessionUsage` and one `model` per session. `PriceMessage` needs each message's own usage and model, which only the engine's transcript read supplies, and D12 names no such read.
- **Trace:**
  - `SessionUsage` holds only totals (`internal/shuttleengine/sessionusage.go:7-20`).
  - `PriceMessage(model, MessageUsage)` prices one message (`internal/shuttleengine/transcriptread.go:143-147`).
  - The per-message model and one-hour cache-write split come from the transcript parse (`internal/shuttleengine/claudeengine/usage.go:604-610`).
  - A session whose forks run on another model, such as `subagent_model`, gets the parent's model in its record. Pricing its totals at that model is wrong, and D12's own reason for per-message pricing does not hold.
- **Fix:** D12 should name the source of per-message usage, the engine's `TranscriptReader` through `shuttleengines.RecordReaders`, and say what the session's `model` field holds when its messages span several models.

**Tool calls**
1. Grepped `SetStatus` and `Revised` in `internal/boardengine/store.go`, to learn whether a status write bumps `revised`, which D8's re-check of the run's own entry depends on.
2. Listed `contracts/recipes/` and darn's rows, to learn whether darn has a Record-Write row and what follows it, for D3.
3. Grepped where `Revised` is stamped (`internal/boardengine/dates.go`), since the first grep did not show the stamping site.
4. Grepped refusals in `internal/beamcli/create.go`, to check that step 4's acting worktree is allowed.
5. Grepped Darn-Write's use of the decision record and darn's recipe rows, for D3's darn walkthrough.
6. Grepped the darn writer stencil for how it treats the record, to settle whether Darn-Write rewrites a seeded part record. It does not.
