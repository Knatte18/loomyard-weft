```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked every pair of decisions in the discussion record against each other, then checked every criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (the D1 beside spec with `KeepViewer`, used by D2's move, against D2's claim that the viewer ends on the beam overview): **broken**, see F1.
- D1 × D2 (the shared sidecar path and build that D2 writes, read by loomcli's `resolveStatusStrandAction`): held.
- D1 × D3 (a recipe `Title` from the seed; an empty or reserved title falls back to the role): held. No recipe in `shedrun.RecipeNames` is named `shell` or `overview`.
- D1 × D4 (both read the recipe from the seed through `shedrun.ReadSeed` on `SelfRunID`; D4 draws no line without a seed): held.
- D2 case 1 × D1 `statusStrandAddSpec` under a concurrent `lym loom start` (overview spec while no runs strand exists yet): held. D2's second failure bullet names the outcome and `lym beam start`.
- D2 case 2 × D3 (a dead `loom-status` relaunched with the beside spec takes the recipe window name): held.
- D2 case 3 × the D2 sidecar write (a live `loom-status` beside a runs strand is left alone and nothing is written): held.
- D2 failure fallback × the D2 sidecar write (`loom-status` gone, sidecar names a dead GUID, the next `lym loom start` adds the strand): held.
- D2 failure fallback × D4 (the overview line names a window that the fallback left absent): held. The support log rejects this as a warned failure with a way forward, and I accept that rejection.
- D3 × Scope Out "a normal task run keeps the overview window" (D3 changes reed for every own-window strand): held. The only `Display.Title` setter today is `internal/orchcli/runs.go:44`, an overview strand.
- D3 break-out path × D1 (`loom-status` never sits in the shell window): held, though nothing in D1 or D2 drives that path.
- D4 × its criterion ("live" against "not done"): **broken**, see F2.
- Constraint `PATTERN-test-speed` × the viewer criterion: **broken**, folded into F1.
- Constraint `PATTERN-no-hard-coded-choice` (placement keyed on the runs strand) × D1 and D2: held.
- Criteria 1–8, 10 and Review × D1–D3: held. Criterion 4's split into "fails before the removal" and "fails after the removal" matches reed: `internal/reedengine/replace.go:45-57` refuses before removing, and `:65-85` adds after removing.

**Findings**

**F1. BLOCKING, design, D2 viewer bullet and its criterion "After the move the viewer is on the beam overview window" (`_lym/discussion/decision-record.md:53`, `:104`).**
D1's own-window `KeepViewer` beside spec, run through D2's ordering (replace `loom-status` first, then add the runs strand), cannot leave the viewer on the beam overview. Nothing in either decision selects it.

The reed code shows why:
- **The replace moves the viewer away.** When the replacement is not an overview strand, `ReplaceStrand` calls `killStrandPanes(..., selectOnEnd=!replacingWithOverview)` (`internal/reedengine/replace.go:70-71`). A viewer on the old overview window is then moved to `relevantWindow`: the newest live agent window, else the overview, else none (`internal/reedengine/ownwindow.go:456-471`). At that moment no overview window exists any more.
- **The replace does not select afterwards.** The post-add select runs only when `endedCurrent && replacingWithOverview` (`replace.go:97-98`).
- **The new checklist window never takes the viewer.** `KeepViewer` means it is never selected (`internal/reedengine/ownwindow.go:479-480`).
- **Adding the runs strand selects nothing.** `AddStrand` of an overview strand never selects its window: "An overview window is never selected here" (`internal/reedengine/spawn.go` doc above `openOwnWindowLocked`).

"As after today's replace" therefore does not carry over. Today the select happens because the replacement is itself an overview strand.

The criterion also conflicts with the record's `PATTERN-test-speed` constraint. Viewer selection is reed/tmux behaviour, but that constraint places every criterion except window naming in pure or fake-seam tests.

Fix: D2 names the mechanism that puts the viewer on the beam overview after the add, and its bound. Options are a select step in `ensureRunsStrand` through a reed op, or adding the runs strand before the move. Otherwise drop the claim and the criterion.

**F2. LOW, consistency, the D4 criterion (`_lym/discussion/decision-record.md:109`) against D4 (`:72-74`).**
D4 draws the line while the run "has a seed and a status file that is not done". The criterion says "while the worktree's run is live". A halted or paused pawl run is not done but arguably not live, so the two state different conditions for the same line.

Fix: word the criterion with D4's condition.

Tool calls: none. Everything I needed was already in the inherited context.
