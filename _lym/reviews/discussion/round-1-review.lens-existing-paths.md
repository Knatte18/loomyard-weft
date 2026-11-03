```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every state, setting and file the record's decisions introduce or change against the code that already writes, resets or reads it.

**Paths traced, per decision**

- D1/D2 (block row, expansion, `on_done` rewrite, skip target):
  - `shedbuild.Parse` / `checkRowShape` (parse.go:109-126) refuses an empty engine. D1 changes Parse for block rows, and Expand stamps `Block` after Parse. Survives.
  - `shedcheck.Check` (check.go:80-115) reads `OnDone` and terminals. Inlined empty `on_done` values are rewritten before any check runs. Survives.
  - `checkLoomContract` / `onDonePath` (loomcontract.go:84-103) walk `OnDone`. The block row's `on_done` is the sub-recipe's entry, and the Bouncer's empty one is rewritten to Drift-Baseline, so the record row comes before the build row on the path. Survives.
- D3 (Block producer, skip target, goto bypass):
  - `enter` (entry.go:52-70) routes a skip by `OnDone` in several places. The plan-writer note covers it, and D3 states the rule. Survives if implemented.
  - `routedState` / pause_after on a skip (run.go:826-835; entry.go:58-66) reads `def.Name`, so it is unaffected. The pause target follows the routing change. Survives.
  - `Goto` (goto.go:122-124) sets `EnteredRow` to the target, so a goto onto the block row bypasses its gate, as D3 says. Survives.
  - `Relaunch` (goto.go:243-244) clears `EnteredRow`, so a relaunch onto the block row evaluates the gate again. This agrees with D3, which limits the bypass to a goto. Survives.
  - Crash after `markEntered` and before the Block producer's Done is persisted: on resume, the step re-calls the Block producer, which returns Done again. Survives.
  - `gotoAdmitted` (goto.go:316-318) reads a skipped entry's route as `def.OnDone`. Breaks (finding 1).
  - `settledSegmentGuard` / `segmentWriter` (loomcli/settledguard.go:108-128) finds the writer by `OnDone == bouncer` and, if the writer was skipped, steps to the row whose `OnDone` names it. In `arras` that is Cartoon, since its `OnDone` becomes Discussion-Write. Discussion-Write carries no gate, so that step is never taken. Survives.
- D4 (halt inside the block):
  - Stuck with an empty `OnStuck` halts at the inlined row (run.go). On resume, `EnteredRow` and the pause checks are keyed by row name, which stays unchanged. Survives.
- D6 (history `block` field):
  - `appendOrFold` / `foldsInto` (run.go:699-719) fold only budget-exempt Stucks of the same producer. Every entry of one row carries the same block path. Survives.
  - `RenameProducers` (rename.go:62,79) rewrites row names only. Survives.
  - History seeding outside shedengine (loomshed/seed.go:58, battencli/arm.go:780) starts with an empty history. Survives.
- D7 (progress, entry numbers, budgets):
  - `mainLine` / `stepIndex` (progress.go:35-84) follow `OnDone`. On the expanded rows the line runs Preflight → Cartoon → Discussion-Write → Discussion-Review → Drift-Baseline. After a pass-over the current row is the exit, so progress jumps there. Survives.
  - `EntryNumbers` (names.go:132-157) gives the non-Bouncer Cartoon row a number of its own. Survives.
  - `segmentBudgets` / `applyReviewBudget` / `stampZeroRounds` (loomrecipe.go:71-88) key on segment name, which stays unchanged. This holds once D11 has them read the expanded rows. Survives.
- D8 (`profile_recipe`):
  - `prepareRunSettings` (battencli/runsettings.go:27-35) is the only reader of `profiles.Default` / `Settings`, and it keys on `settings.Recipe`. D8 routes that through one accessor. Survives.
  - `checkProfiles` (configreg/runsettings.go:162-176) validates the file's own blocks and does not key on a task's recipe. Survives.
  - `boardcli` profile-name check (cli.go:53) checks only the profile's name. Survives.
- D9/D10/D11 (embedded recipes, accessors):
  - `BouncerRunSubdir` (bouncerrows.go:18-38) parses tapestry and darn and finds the Bouncer by row name. `arras`'s Discussion-Bouncer resolves through tapestry's copy of the same row. Survives.
  - `RecipeEngines` and the shedrecipe coverage guard (coverage_guard_test.go:38-65): `Block` appears only after expansion. D11 plus the plan-writer note route these through the expanded parse. Survives.
  - `reportengine` `runRecipes` / `recipePhaseTables` (runs.go:24, timingspans.go:70): D10 names the run reports. Survives.
  - `childRecipeSupported` (battencli/wire.go:532): D10 names Seed-Child. Survives.

**Findings**

1. LOW · design · D3 (skip target) — `gotoAdmitted` reads a skipped entry's route as `OnDone`, which for a passed-over block row is the sub-recipe's entry, not the skip target the run actually went to.
   - Evidence: internal/shedengine/goto.go:316-318 (`case Done, OutcomeSkipped: routed = def.OnDone`). After D2 the block row's `OnDone` is the sub-recipe's `entry`.
   - Effect: when `current_producer` names no row and the latest named entry is the block row's skipped entry, the goto admission takes the block's first inlined row as the run's routed row. It admits a target inside a block the run passed over, and does not admit the exit.
   - D3 says "a skipped row routes to it [the skip target]". It is silent on readers that reconstruct a skip's route from the history, and the plan-writer note names only `entry.go`. D3 should state that every reader that reconstructs where a skip routed reads the skip target.

Tool calls:
1. grep for `HistoryEntry{` outside shedengine, `.OnDone` readers outside shed, and loomcli goto guards: to find other writers of history and other readers of `OnDone` the manifest lacked.
2. Read `loomcli/settledguard.go` lines 37-140: a goto guard that reads `OnDone` and skipped entries, one hop past the manifest.
3. Read `bouncerrows.go`, grep `beamcli/dobbydeps.go` and the shedrecipe coverage guard: embedded-recipe readers keyed by row and engine that the manifest named but did not show.
4. grep for index-addressed rows in loomengine/loomcli/loomshed, and read `appendOrFold`/`foldsInto`: to check whether inlined rows shift index-keyed consumers, and how history folding treats the new field.
