```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I traced each decision's mechanism step by step on one concrete case and checked the end state against the decision's stated purpose.

**Per decision**
- **D1.** Case: Resume after a reed server restart. The strand table lists loom-status (key 1) before runs (key 0). Loom-status relaunches first, finds no overview window and opens before the lowest-index window. Runs relaunches next. No overview window has a key no higher than 0, so it opens before the lowest-index window, ahead of loom-status. Order: runs, then loom-status. **Met.**
- **D1, replace.** `ReplaceStrand` removes the old strand and kills its window before the add, so placement reads only the windows that remain. **Met.**
- **D2.** Case: two overview windows, no live agent. `followTargetWindow` keeps the last overview line it sees and `agentWindowPanes` keeps the last overview pane, so today each returns one arbitrary overview. D2 replaces that with the lowest-index one. **Met in intent; one wording gap, F2.**
- **D3.** Case: a beam created after the change. The first pass runs `lym loom start` (battencli wire.go). `armAt` resolves `c.recipe` from the seed before the start body runs. The seed is written by the batten Seed-Child, so the recipe reads `pawl`. The add spec gets key 1 and Title `pawl`, and the runs strand added later at key 0 opens before it. **Met**, with the residual in F3.
- **D4.** Case: `lym beam start` with loom-status live and no runs strand. The add is no longer refused because D1 removes the refusal, and loom-status is left alone. **Met.**
- **D5.** Case: a legacy session whose runs strand replaced loom-status. The heal needs a new `lym loom start` in the beam. **Partly met**, see F1.

**Findings**
- **F1 — MEDIUM — D5.** The heal rests on the pawl run's "next `lym loom start`", but the record never says when that happens for a pawl run sitting at Beam-Await. The only caller found is the batten pass (`exec.Command(exe, "loom", "start", "--no-attach")` in `internal/battencli/wire.go`). If dobby runs no further pass while the driver lives, the legacy session keeps no checklist window until a hand `lym loom start`. That is an unresearched fact the bound depends on. Fix: state who runs the next start, or name `lym loom start` in the beam as the way forward.
- **F2 — LOW — D2.** "The lowest-index window carrying the overview marker" fits `followTargetWindow`, which reads tmux options. `relevantWindow` goes through `agentWindowPanes`, which identifies the overview from reed state (`isOverview(display)`), not from the marker, and has no window index until it is matched against the window listing. An overview window opened by an older lym carries no marker, so the two rules can disagree on legacy windows. Fix: word D2 per source (reed state for `relevantWindow`, the marker for follow), with the index taken from the window listing.
- **F3 — LOW — D3.** `resolveSeedRecipe` returns `tapestry` when no seed exists. Any status strand added by a `lym loom start` that runs before its run's seed is written gets the Title `tapestry`. The record does not say the seed always exists before `ensureStatusStrand`. The `seedAndCommitBootstrap` code in start.go seeds in steps 1–3 before step 4, but `c.recipe` was fixed earlier in `armAt`. Fix: have D3 or the plan note take the Title from the seed that `seedAndCommitBootstrap` returns, not from `c.recipe`.

**Tool calls**
1. grep of `internal/loomcli/start.go` for step order and the `ensureStatusStrand` call: needed to know whether the recipe is resolved before the status strand is ensured.
2. grep for `overviewWindowInsertArgs`/`isOverview(` callers in reedengine: needed every placement and fallback site for D1 and D2.
3. Read of `internal/loomcli/start.go` (lines 470–530) and `resolveRecipe` in `internal/loomcli/arm.go`: needed the start body's order around seeding.
4. Read of `agentWindowPanes` and `followTargetWindow`: needed the fallback mechanics for D2.
5. Read of the pawl run start in `internal/beamcli/create.go` and of `resolveSeedRecipe`: needed what recipe an unseeded start reads.
6. grep for passes that run `lym loom start` (battencli, dobbyengine): needed for D5's heal trigger.
