```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked the record's claims, bounds and ordering promises.

**Findings**

1. **MEDIUM, D3 and Notes for the plan writer.** The claim that the window gets "the recipe name the run's seed records" can break on a fresh start, and the record hands this fact to Plan instead of settling it.
   - Evidence: `loomCLI.resolveRecipe` in `internal/loomcli/arm.go` runs in the persistent pre-run. When the run has no seed yet it sets `c.recipe` to `shedrun.RecipeTapestry`.
   - `lym loom start` writes the seed only later, in `seedAndCommitBootstrap` (the start.go RunE, before step 4). It does not re-read the recipe afterwards.
   - Counterexample: a run whose first `lym loom start` seeds it with a recipe other than tapestry gets its checklist window titled `tapestry`.
   - The notes bullet "check that the start path reaches it" leaves this to Plan, which the rubric's "deferred research question" bullet flags.
   - Fix: make D3 say the title is the recipe read after the seed is written, not the pre-run value. Or confirm that every start which reaches `ensureStatusStrand` already has a seed, and record that with evidence.

2. **LOW, D1 bound.** The bound says every overview spawner adds under an explicit role that `AddStrand` refuses while it is held. That holds, but it leaves out something that also bounds the cap.
   - Evidence: `strandNameLocked` refuses a `NameOverride` that is already held, and both spawners set `NameOverride`.
   - A plain `Role` add would get a number (`x-2`) instead of a refusal. What actually stops that is `internal/reedcli/add.go`, which rejects `--anchor overview` as engine-only.
   - The bound should name that the CLI cannot add an overview strand, so only engine callers (the runs strand and the loom status strand) can.

3. **LOW, D1.** The record does not say whether `UpdateStrand` may change an overview strand's order key in place. `refuseWindowMove` covers anchor and group changes only, so an update could leave a window out of order until its next relaunch. The record should say whether the key can be updated, and what happens to the window if it can.

**Claims I attacked**

- D1, ties placed in opening order, with a legacy key-0 status strand and a runs strand replaced at a build change: held. The runs window relaunches after the legacy one, which D5's bound covers.
- D1, removing the refusal while the explicit role still blocks duplicates: held for `NameOverride` callers. The CLI path is closed by `reedcli/add.go` (finding 2).
- D1, replacing an own-window status strand with an overview one: held. `ReplaceStrand` is a remove and an add, and `refuseWindowMove` only applies to updates.
- D2, lowest-index overview as the fallback: held. `followTargetWindow`'s listing carries `window_index`, and a legacy unmarked overview window counts as an agent window, as it does today.
- D3, "loomcli already imports orchcli": held (`bootstrap.go` imports it).
- D3, recipe name on a fresh start: broken (finding 1).
- D4, `lym loom start` and `lym beam start` running at the same time: held. The two adds no longer depend on each other, and each runs under reed's op lock.
- D5, the next loom start heals a session: held. A sidecar from another build forces a replace, and a missing status strand gets an add. The bound admits the wait until that next start.
- Scope claim that `lym beam create` starts the pawl run before `lym beam start`: held.

**Tool calls**

- Read `strandNameLocked` and grepped loom start: needed to know whether `NameOverride` refuses or numbers, and where the recipe is set.
- Grepped `armAt` and `AnchorOverview` callers in start.go and reedcli: needed to find the recipe read and any hand path that adds an overview strand.
- Read start.go RunE lines 498–530 and the reed add anchor flag: needed the order of seeding against the status strand step.
- Read `resolveRecipe` in arm.go: needed what `c.recipe` holds before the seed exists.
