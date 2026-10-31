```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's stated purpose.

**Findings**

1. **BLOCKING — D7 (and Scope's "pawl out"):** removing the board-entry snapshot and `internal/darnengine/snapshot.go` breaks pawl.
   - Case: a pawl run reaches its Code-Review perch.
   - Steps: `internal/loomcli/pawlwiring.go:135` calls `writeBoardEntrySnapshot`, which is defined in `internal/loomcli/darnwiring.go:199`. That function calls `darnengine.BoardEntrySnapshot`, which writes `_lym/discussion/board-entry.md`. Pawl's perch reads that file as its answer key, and pawl has no decision record.
   - If D7 removes the snapshot writer, pawl's perch loses its answer key. That contradicts "this task does not touch pawl" and D2's promise that pawl is served as today.
   - D7 needs to keep the snapshot for pawl until `pawl-loom-recipe` lands, or say what replaces it.

2. **MEDIUM — D6 with D4/D5:** the darn writer's no-parent stop no longer halts the run.
   - Case: a darn run with no parent, whose entry is outdated.
   - Today the stencil says: end the turn with the stop report "so the gate exhausts and halts the run". That gate is the Darn-Write `discussion` gate. It fails because the writer has not written a passing record.
   - Under D4, Record-Write has already written a valid record, and D5 makes the writer's discussion gate validate-only. That gate passes.
   - The verify gate on an empty diff and the description gate on the stop report can also pass. The run would then go on to Drift-Verify, Code-Review, Describe and Finalize with no change, where it should halt.
   - D6 says "stops with a stop report … as today", but the mechanism that stopped it is gone. D6 needs a halt that does not depend on the record.

3. **LOW — D4 and the Drift rows:** darn's Drift-Baseline still names the board entry as its subject.
   - `shedrecipe/entries_drift.go` accepts `decision-record` or `board-entry` as a row subject.
   - Darn's Drift-Baseline ran before any record existed, so it uses `board-entry`. Under D4 the record now exists before Drift-Baseline.
   - The record is silent on whether darn's Drift rows switch to `decision-record`. This is not a breakage, but it is a loose end next to D7's "the record is the answer key".

4. **LOW — D4 ordering:** `lym loom decision add` on a darn run before Record-Write runs finds no record.
   - `AppendDecisions` needs a `## Decisions` heading, so it errors in the window between seeding and the row.
   - That window is short (Preflight only), but the record states no bound or refusal message for it.

**Checked, held:**
- D1: tapestry has DiscussionWrite → Implementer → Finalize → Friction-Reflect, and darn has RecordWrite → DarnWrite → Finalize. Both pass. A recipe with no record row is refused.
- D2: a pawl run started by `lym loom start` from Run-Shed passes the named exception. An unseeded worktree reads as tapestry.
- D3: `insert-card` on darn has no Plan-Write row, so it refuses. `peers` reads a darn peer's own seed and gets the darn rows.
- D5: the D4 record (`## Goal` filled, other sections empty) has no empty-section finding and no decision ids, so it passes. A tapestry record whose fixer drops an Added entry is caught by the history check.
- D8: `verify_source: tree_scope` reads `darn.yaml`'s `tree_scope`. The single Describe stencil renders absent run records as `(none)`.
- D9: tapestry parked on Drift-Verify resumes, then the perch runs, then Describe. Purpose met.
- D10: tapestry Preflight with a broken `batcher.yaml` fails early. Darn and pawl skip the check. A run parked on Tapestry-Preflight maps to Discussion-Write.
- D11: a seed recording `beam` is refused as an unknown recipe.
- D12: a hub `loom.yaml` with `plan_max_bounces` moves to `tapestry.yaml` through KeyMoves. In run_settings, `retiredByKeyMove` refuses only a module that is no longer registered, and `loom` stays registered, so a `loom` object is not refused.
- D13: removed keys are retired, and values are read from `shed.yaml` and the profile block. Purpose met.
- D14: nothing to execute.

**Tool calls:**
- One grep of `board-entry`/`BoardEntrySnapshot` readers, `configreg/runsettings.go` lines 35–100, and `darn-recipe.yaml` lines 85–110. My context lacked the snapshot's callers beyond darn, how key moves treat run_settings, and the Darn-Write gate config.
