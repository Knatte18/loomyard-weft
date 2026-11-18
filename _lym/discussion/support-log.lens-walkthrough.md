```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end on one concrete case.

**Cases walked**
- **D1** (shared spec). Case: a pawl run's `lym loom start` in a beam session that already holds `runs`. `statusStrandAddSpec` finds `runs` and builds the beside spec with `Title` set to `c.recipe`, which `resolveRecipe` reads from the seed. orchcli's move builds the same spec. Purpose met.
- **D2** (move, happy path). Case: beam `B` and pawl run `self`. The session holds `loom-status` in the overview window and no `runs`. Step 1, `ReplaceStrand(loomStatus.GUID, beside spec)`: reed removes the old strand, kills its pane and ends the overview window. `addStrandLocked` then opens an own window, and `validateDisplay`/`render.ValidateDisplay` accept a `Title` on an own-window strand. Step 2 writes the sidecar under `LoomScratchDir(c.location)`; `c.location` is beam `B`, since `pawlwiring` calls `BeamUp(location)`. Step 3, `AddStrand(runs)`: no overview strand is left, so reed accepts it. The next `lym loom start` reads the sidecar and keeps the strand. Purpose met.
- **D2** (failure path). Case: step 1's add fails after the removal. Purpose not met; see finding 1.
- **D3** (window name). Case: a `loom-status` own-window strand with `Title` `pawl` against a shuttle agent strand with an empty `Title`. No current own-window caller sets `Title`, so no existing window gets renamed. Purpose met, provided the break-out path is covered, which the record does name.
- **D4** (overview line). Case: beam `B` with its pawl run in Beam-Await. `ReadSeed(B, self)` gives recipe `pawl`, and the status poll gives row Beam-Await, state running. The line reads `pawl  Beam-Await  running  window pawl`. After Beam-Down the status is done and the line goes. Purpose met.

**Findings**

1. MEDIUM, D2 failure fallback. The fallback is handed a strand that may no longer exist.
   - D2 says that if step 1 fails it "falls back to today's replace of `loom-status` by the runs strand".
   - `ReplaceStrand` (`internal/reedengine/replace.go`) fails in one of two states:
     - It fails in `validateNaming`/`validateDisplay`/`requireSessionLocked` before any removal. `loom-status` then stays in the overview window, so today's replace works.
     - It fails in `addStrandLocked` after `removeStrandLocked` and `killStrandPanes`. The strand is gone and the removal is saved; the comment there says "the reconcile tail below persists the removal alone".
   - In the second state, a replace on `loomStatus.GUID` gets a GUID that no longer exists. It fails, so the beam ends up with neither the checklist nor the overview.
   - The fix is to read the strands again after a failed move: replace when `loom-status` is still there, add `runs` when it is gone. The failure-path acceptance criterion should cover both states.

2. LOW, D2 trigger and Goal "always". `ensureRunsStrand` handles only the case where `loom-status` exists and `runs` does not.
   - After a reed server restart both strands are dead. `lym beam start` then hits the `runs != nil` branch and replaces only `runs`. Nothing brings the dead `loom-status` back until a `lym loom start` in the beam's worktree.
   - The only Open risk names "a session started on an older build". This path happens on the current build and is not recorded, while the Goal says the session shows the checklist "while a beam's pawl run lives".
   - Fix: narrow the Goal's claim, or widen that Open risk to cover any lost checklist strand.

3. LOW, D2 repeat move. If `runs` goes missing while `loom-status` already sits in its own window, the next `ensureRunsStrand` runs the move again.
   - That kills and relaunches the checklist pane, an own window replaced by an own window, and rewrites the sidecar. No harm, but the record does not say the move is meant to run in this case.

Tool calls beyond my context:
- Read `ReplaceStrand` in `internal/reedengine/replace.go`: I needed reed's state after a failed replace to walk D2's fallback.
- Read `validateDisplay`/`isOverview` and checked when `resolveRecipe` runs in `internal/loomcli/arm.go`: to confirm reed accepts `Title` on an own window and that `c.recipe` is set from the seed.
- Read `render.ValidateDisplay` and `killStrandPanes`: to confirm a `Title` passes validation and to see how panes are killed on the move.
