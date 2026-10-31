```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the state each decision changes through the code paths that already write or read it.

**Paths traced, per decision**
- D4: `lym loom decision add` on a darn run (`internal/loomcli/decision.go`) calls `discussionparser.AppendDecisions(recordPath, supportLogPath, ds)`, and that runs `Validate(decisionRecordPath, supportLogPath)`. **Breaks.**
- D4: the drift subject (darn's Drift-Baseline and Drift-Verify rows use `subject: board-entry`). **Breaks.** The record says nothing about it.
- D4: resume or goto back to Record-Write. **Survives.** The row leaves an existing record alone.
- D5: `validate.go` picks the darn check by `c.env.DiscussionHistory != nil`. **Survives** if the choice is rewired to follow the record row's engine. The `AppendDecisions` validate path breaks, as in the first D4 line.
- D7: pawl's `WriteSnapshot` in `internal/loomcli/pawlwiring.go:135` calls `writeBoardEntrySnapshot`, which uses `darnengine.BoardEntrySnapshot`. **Breaks.**
- D7: darn's snapshot writer in `darnwiring.go:190`. **Survives** (it is removed as intended).
- D9: Finalize `on_reroute: Parent-Align` in tapestry now passes through Drift-Verify again before the perch. **Unclear.** The record does not say whether Drift-Verify tolerates a second pass.
- D10: goto, resume and retired-row mapping onto Tapestry-Preflight. **Survives** through the retired-row map.
- D12: `KeyMoves` rewrites both the hub base files and run_settings (`moveRunSettings`). **Survives.**
- D12: the row setting references `reviewer_fan_setting` and `auto_accept_setting` (`loom.plan_*`). **Survives** only if they are renamed too. D12 names only `entry_gate` and `implementer_setting`.
- D12: the fallback from `plan_max_bounces` to `review_max_bounces`. **Breaks**, because it now crosses modules.
- D13: `loomengine.RecipeOverride` sends darn to `darn.yaml` and every other recipe, pawl included, to `loom.yaml`. **Breaks quietly.**
- D13: a task's run_settings or profile carrying `loom.driver_kind` or `darn.driver_kind`. **Breaks quietly.**
- D11: `decodeSeed` and `ParseSeed`. **Survives.**

**Findings**

1. **BLOCKING, D4/D5:** `lym loom decision add` stops working on every darn run.
   - D4 writes no support log.
   - `AppendDecisions` validates through `discussionparser.Validate(decisionRecordPath, supportLogPath)`, which reports `discussion-file-missing` when the support log is absent.
   - The verb also passes `supportLogPath := loomengine.DiscussionSupportLog(target)` unconditionally (`decision.go:784`, `:806`).
   - D5 makes the support-log check depend on the record row's engine, but the record never routes `decision add`'s own validation through that choice.
   - D6 depends on `decision add` working on darn, so the outdated-entry route fails with it.

2. **BLOCKING, D7:** removing `internal/darnengine/snapshot.go` breaks pawl.
   - `pawlwiring.go:135`'s `WriteSnapshot` writes and commits `board-entry.md` through the same `writeBoardEntrySnapshot` and `darnengine.BoardEntrySnapshot`, as pawl's Code-Review answer key.
   - `darnwiring.go:196`'s doc names this directly: "a darn run's task, and a pawl run's beam project".
   - The record puts pawl out of scope but removes code pawl reads.

3. **MEDIUM, D4/D7:** darn's Drift-Baseline and Drift-Verify still read the subject from the live board entry.
   - The rows have `subject: board-entry` (`darn-recipe.yaml:78`, `:116`). `driftengine/doc.go:59` says this reads the board entry in a darn run.
   - Once darn has a mechanical record, D7 makes the record the decided scope, but Drift keeps reading the board.
   - The record never says whether those rows switch to `subject: decision-record`, as tapestry's rows use.

4. **MEDIUM, D13:** the record does not cover two paths that read the removed keys.
   - `loomengine.RecipeOverride` (`config.go:~1152`) loads `loom.yaml` for every non-darn recipe, pawl included, and `ValidateLocalValues` runs `sheddriver.CheckLocalOverride("loom", …)`. Once the keys are retired, the override has nothing to read, and the record does not say the function goes or what pawl reads afterwards.
   - An existing task's `run_settings` holding `loom.driver_kind`, `darn.driver_kind` or `arbiter*` would be either refused (unknown key) or silently ignored (retired with `CategoryNone`). Ignored means a task pinned to the llm driver silently runs go.
   - The record leaves profile repair to the orch but says nothing about run_settings on live entries.

5. **MEDIUM, D12:** the record does not say how `tapestry.plan_max_bounces: null` reaches `loom.review_max_bounces`.
   - Per-perch budgets fall back to `review_max_bounces` within one module today (`loomengine/template.yaml` line 49, "null takes review_max_bounces"; `loomrecipe.segmentBudgets`).
   - After the split, the fallback spans two modules.

6. **LOW, D12:** the list of renamed setting references is too narrow.
   - Rows also carry `reviewer_fan_setting: loom.plan_reviewer_fan`, `auto_accept_setting: loom.plan_auto_accept`, and `budget.Plan-Review` budgets.
   - D12 names only `entry_gate` and `implementer_setting`.
   - These are not call sites: they are the method by which a moved key is read.

7. **LOW, D9:** the reroute path now re-runs Drift-Verify.
   - Today tapestry's Finalize reroutes to Parent-Align and then goes straight to the perch.
   - Under D9, Parent-Align leads to Drift-Verify first, so a rerouted run runs Drift-Verify a second time.
   - The record does not say that a second Drift-Verify pass on the same baseline is valid.

**Calls made**
1. Searched for board-entry snapshot writers and readers, and read `RecipeOverride` in `internal/loomengine/config.go`. My context showed neither who else writes the snapshot nor how the overrides are chosen per recipe.
2. Read the drift `subject:` keys in the recipes, the run-settings handling in `internal/configreg/runsettings.go`, and pawl's `WriteSnapshot` in `internal/loomcli/pawlwiring.go`. These were needed to trace D4/D7 against Drift and pawl, and D13 against run_settings.
3. Read how `AppendDecisions` and `decision.go` use the support log, and how driftengine reads the board-entry subject. These were needed to check whether `decision add` survives D4's missing support log.
