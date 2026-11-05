```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: tracing the code that already runs and touches state D1–D14 change, and saying whether each decision's result survives it.

**Findings**

- **BLOCKING — D9/D10: the run's own status writes undo Raddle-File's status clear.**
  D9 clears the run status inside Raddle-File's board write. But the loom commit-status seam (`newCommitStatusSeam` in `internal/loomcli/wiring.go`) writes `<state> · <producer>` to the board on every later transition through `Board.SetRunStatus`, including Raddle-File's own Done and the run's finish. `SetRunStatus` (`internal/boardengine/board.go:200`) skips only an absent or done entry.
  - `tapestry` shape: the released task gets a run-form status back. Dobby will not start it, and the run lock now holds it.
  - `project` / `beam` shape: the project gets a run status.
  - D10 removes the end-as-Sketch finish, which is what clears that status for motif today. After D10, nothing clears it.
  - Motif avoids this because `MakeSketch` runs in `Finish`, after the last status write.
  - Fix: do the filing write in the run's `Finish` hook, as `sketchFinish` does. Or have the status seam skip an entry the run no longer holds.

- **BLOCKING — D9 `tasks` shape and D14: Land prunes the done Sketch together with its record.**
  - batten's Land row always calls `PruneBoard` → `board.Prune(true)` (`internal/battencli/land.go`, about lines 114-122). Prune removes every done entry that carries no `notify_on_land`.
  - `internal/boardengine/decision.go:104-116` deletes the `decision-<slug>.md` of every removed entry.
  - So a Sketch that D9 marks done, "keeping its record", is removed with its record by the same run's Land, minutes later. The README link is gone too.
  - The stored `raddle-<slug>.yaml` either orphans or must follow the same removal. Either way, D14's `lym report raddle` loses the stored proposal for every `tasks`-shape run. D11's `--proposal` read is safe, since it applies only to `beam`.
  - Fix: keep the entry, e.g. as a Sketch marked raddled, or store the proposal where prune does not reach. Then say which.

- **MEDIUM — D2: `Board.Promote` cannot carry a recipe, and a promoted Sketch is released at once.**
  - `Store.Promote` (`internal/boardengine/store.go:759`) only sets `Kind = task` and clears priority. It keeps labels, takes no recipe, and leaves `Status` nil, because `sketchOf` already cleared it. A nil status reads as released.
  - "Promote with recipe `raddle`" is therefore two writes: promote, then upsert the recipe.
  - Between them, either D2's new label rule refuses the promote itself (recipe empty ≠ `raddle`), so a Sketch can never be promoted, or dobby may start the entry under `RecipeOf`'s board `default_recipe`.
  - D2 must say whether promote gains a recipe argument or the rule judges the explicit `recipe` field, and that the entry is never released without it.

- **MEDIUM — D12: sessions that never finish Done record no usage.**
  - `shuttleengine` fills `Result.Usage` only for `OutcomeDone` (`internal/shuttleengine/wait.go:1931`).
  - Sessions that halt, go stuck, are killed, or are interrupted then resumed keep their tokens in transcripts, but no usage file is written. The runs with the most trouble are understated in `lym report cost`'s fit.
  - The record is silent on this. It should state the bound, or read usage at every session end.

- **LOW — D12: the per-session file key must be the session id.**
  `SessionUsage` sums a whole transcript, so a resumed session that finishes again must overwrite its earlier file, not add a second one. D12 says "one file per session" but names no key.

- **LOW — D8: any weighed peer that lands fails the re-check.**
  A weighed peer that lands and is pruned leaves `board_checked`, so the re-check fails. With busy boards, auto mode may spend its 2 returns on ordinary landings. This may be intended, but the record does not say so.

**Paths traced, per decision**

- D1: `childRecipeSupported` / `boardRecipe` — survives, given the additions. `childFinalized` reads a raddle run as "no code landed", which is correct.
- D2: `Store.Promote`, `sketchOf`, `RecipeOf` default — breaks (finding 3).
- D3: tapestry's Motif-block `seedDiscussedRecord` path — survives, since tapestry has no Record-Write row. Darn's Record-Write — behaviour changes, as the record already states.
- D7: sampler-done analogue — survives.
- D8: `revised` stamp under status writes — survives. Running entries' `revised` stays put while their status changes, e.g. `pawl-orch-landing` shows revised 06:00 while its status moved later. Pruned peers — finding 6.
- D9: run lock exemption — survives as designed. Commit-status seam — breaks (finding 1). Land's `PruneBoard` with decision-file removal — breaks for `tasks` (finding 2). Prune stripping depends_on — survives.
- D10: `finishesAsSketch` replaced by a row check — survives. `ClearMotifRun`'s run-lock and status-lock wait — survives: the start waits for the pair.
- D11: `NotifyOrchAt` with no strand posts nothing — the premise holds.
- D12: `wait.go` usage only on Done — partial (finding 4).
- D13 / D14: archive-history reads — survive. D14's stored proposal under prune — breaks (finding 2).

**Tool calls**

1. `land.go` lines 60-175 and `Board.Promote`: I needed Land's board steps and the promote semantics, which my context lacked.
2. `Store.Promote`, prune's removal paths and `RecipeOf`: I needed promote's field handling and what prune deletes.
3. `decision.go` lines 100-120 and `board.go` lines 466-482: to confirm that removing an entry deletes its decision record.
4. `Board.SetRunStatus`: to see whether a later status write skips an entry the run no longer holds.
5. `Revised` in `store.go` / `board.go`: to check whether status writes bump `revised` (inconclusive; settled from `board list` data).
