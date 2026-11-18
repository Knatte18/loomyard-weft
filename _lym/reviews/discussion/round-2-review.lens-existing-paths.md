```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens review of the Hardener decision record (round 2).

**Paths traced, per decision**

- D1: `loomrecipe` `verifyGatedRows` (`internal/loomrecipe/darnrecipe.go:30-50`) adds `HardenFix` to the verify-budget rows once it joins `LoomBuildEngines`. **Survives**, provided the arm reads `verify_attempts` from `hardener.yaml` as D1 says. `shedbuild` `recordOnly` and `onlyRowOf` read the same list (`loomcontract.go:34,59`). **Survives.**
- D2/D9: task `run_settings` go through `configreg.PrepareRunSettings` and `checkLocalDocument` into the pair's local layer. That layer holds only `CategoryNone` keys (`configengine/doc.go:103`). **Breaks** for every guard-class key D9 lets `run_settings` override (finding 1).
- D2: Preflight checks that "the hub's own `gate.yaml` sets `harden-live`". This meets the sparse-base strip and the template fill of `gates` entries. **Breaks** at one edge (finding 4).
- D3: `loomCLI.reviewBudget` and `loomSegmentBudgets` (`wiring.go:1424`) are named in D3. **Survives as designed.**
- D4 `--copies`: `gatecli` runs each tier through its own `acquireSlot` → `AcquireSlots` with the tier's slot count (`run.go:737`). Inheritance applies only through the `LYM_GATE_SLOT` env var (`run.go:149`). **Breaks.** The record does not say how the call's own n-slot hold relates to that per-tier acquire (finding 2).
- D5: `lym tender reap` skips processes working inside a registered worktree (`tenderengine/reap.go:36-37`), so it does not race the sandbox reaper. **Survives.** `lymtemp` sweep: the root is not under `lymtemp`. **Survives.** The committed `loom.yaml` override `selfreport: false` is read by no module. **Breaks** (finding 5).
- D6/D7: the Bouncer archives its run directory on a new generation (`bouncer.go:225`). The per-round files live in that directory and are copied by the fix row before any re-entry. **Survives.** `commit_seam` accepts only `plan` or `discussion` (`entries_bouncer.go:139`). D7 extends it. **Survives as designed.**
- D8: `hubreconcile` applies reinterpretations to the hub base file alone (`reinterpret.go:50`). **Breaks** for `efforts` keys held in other layers (finding 3). The walk order, `ReconcileHubWideAt` before `applyReinterpretations` (`walk.go:232,254`), **survives**: binary-change reconciles keep template-lacking keys.
- D10: no existing reader beyond the contract check and `verifyGatedRows`. **Survives.**

**Findings**

1. **BLOCKING**, design. Location: `_lym/discussion/decision-record.md:216` (D9's "A task's `run_settings` may override any of them"), together with D2 at line 49.
   - Problem: `run_settings` become the pair's local layer, which holds only `CategoryNone` keys (`internal/configengine/doc.go:103`; `internal/configreg/runsettings.go`, where `PrepareRunSettings` calls `checkLocalDocument`). PATTERN-config-layers classes as Guard any key that hollows a guard, or that sets the reviewer or lens set.
   - Existing precedent puts D9's keys in Guard:
     - `&lt;perch&gt;_auto_accept` is guard (`internal/reviewbudget/reviewbudget.go:42`).
     - The loom fans are guard (`internal/loomengine/config.go:649-656`).
     - darn's `tree_scope` is guard and "no task run_settings sets it" (`internal/darnengine/config.go:59`).
   - D2's own charter keys meet the same test. `packages` narrows the live gate. `llm_driving: true` turns off the concurrent-copies run.
   - The record classifies none of `hardener.yaml`'s keys. Classing them Guard makes the `run_settings` path refuse D2's charter and Preflight's way forward. Classing them None breaks the category test.

2. **MEDIUM**, design. Location: `_lym/discussion/decision-record.md:107` (the D4 `--copies` paragraph).
   - Problem: today a `lym gate test` call takes slots per tier run, through `acquireSlot` → `pool.AcquireSlots` with the tier's own count, default 1 (`internal/gatecli/run.go:737`; `internal/gateslot/tiers.go:194`). It skips that acquire only when `LYM_GATE_SLOT` names a held slot of an enclosing run (`run.go:149`).
   - D4 has the call itself take n slots and run copies "inside them". It does not say whether the tier run inside that call still makes its own acquire against the pool the call already holds whole (with n = slots = 3, it waits on its own slots), or how n copies inherit n slots. `Lease.InheritEnv` names only the first slot (`internal/gateslot/doc.go:48-49`).

3. **LOW**, design. Location: `_lym/discussion/decision-record.md:195-196` (D8's reinterpretation).
   - Problem: `applyReinterpretations` rewrites only `configengine.ConfigFile(boardDir, module)`, the hub base file (`internal/hubreconcile/reinterpret.go:50`).
   - `efforts` is `CategoryNone` (`internal/agentconfig/config.go:124`), so a pair's local layer, a task's `run_settings` or a run profile can still hold a `treadle-judge/&lt;model&gt;` key. `checkEfforts` refuses that key once the role is gone (`config.go:218`).
   - The criterion "rewritten … and then loads" holds only for the hub file, and the record does not say so.

4. **LOW**, consistency. Location: `_lym/discussion/decision-record.md:54` (D2's Preflight check on the hub's own `harden-live` key).
   - Problem: PATTERN-config-layers' sparse-hub-base rule has `hubreconcile`'s strip drop a template-equal unit. Gate load also fills template keys into `gates` (`internal/gateslot/doc.go:113`).
   - A hub whose live tier is `tier0` sets `harden-live` equal to the template's `[0]`. The strip removes it, and Preflight then refuses a correctly configured hub.

5. **LOW**, design. Location: `_lym/discussion/decision-record.md:126` (D5's committed `loom.yaml`), with Open risks line 285.
   - Problem: no module declares or reads a `selfreport` key. `loomengine.ConfigSpec`'s categories hold none (`internal/loomengine/config.go:633-682`). The only code under `internal/` that files an issue is `lym selfreport create` (`internal/selfreportcli/cli.go:132`). Reflection files issues only through that verb (`internal/friction/doc.go:49`).
   - `selfreport: false` therefore loads with an unknown-key warning (`internal/configengine/doc.go:13`) and stops nothing. Only `friction: ""` turns off the automatic path, so D5's sentence and the Open risk's "the committed override stops only the automatic paths" rest on a key that does not exist.

**Tool calls**

1. `selfreport` key search, `darnrecipe.go:25-50` and the `hubreconcile` walk symbols: the context lacked the `selfreport` config site, the other `LoomBuildEngines` reader and the walk order.
2. `selfreportcli` config grep and `walk.go:225-262`: needed to confirm whether any config key gates self-report, and whether reinterpretation runs after the hub-wide reconcile.
3. Template-wide `selfreport` grep and `configsync` `CheckLoad`: needed to confirm no template declares `selfreport`.
4. `selfreportengine` lines 30-60 and `CreateIssue` callers: needed to find the automatic self-report path.
5. `configengine/doc.go` unknown-key handling: needed to know whether an undeclared `loom.yaml` key is refused or only warned.
