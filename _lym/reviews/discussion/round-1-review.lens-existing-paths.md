```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: tracing the code paths that already write or read the state each decision touches.

**Paths traced, per decision:**
- D1, `verify_source: tree_scope`: `loomCLI.verifySource` (`internal/loomcli/arm.go`, around line 351) sends every tree_scope recipe to `darnVerifySource`. D1 says the arm reads `hardener.yaml` for this, so the change is stated. **Survives.**
- D1/D3, perch maps: `loomPerches` and `loomSegmentPerches` (`internal/loomcli/wiring.go:1405-1419`) are closed maps keyed by Bouncer row and segment. D1 and D3 add the Harden perch and the Bouncer-row lookup. **Survives** when wired.
- D3, lightweight review budget: `loomCLI.reviewBudget` (`arm.go`, after `verifySource`) re-reads only `loom.yaml` and `tapestry.yaml` when the full wiring has not loaded config. `loomSegmentBudgets` reads `cfg.Caps(perch)` from loom config (`wiring.go:1422-1427`). **Breaks** (finding 3).
- D3, burler model lists: `RoundModels.Pick` / `pickRound` (`internal/burlerengine/models.go`) clamp to the last entry. D3's cyclic expansion meets this, and the clamp is pinned by `models.go`. **Survives.**
- D4, nested slots: `lym gate test` help (`internal/gatecli/cli.go`, Long text) says a call inside slots an enclosing gate run holds acquires no others. The `live` gate calling `lym gate test --copies` inside its held slots matches that. **Survives.**
- D5, hub override: the `loom.yaml` `selfreport` key is read by loomengine's config load, which ignores it as retired (`internal/loomengine/config_test.go:212`). Only `friction` turns the automatic path off (`internal/loomengine/template.yaml:37`; `internal/friction/doc.go:49`). **Breaks** (finding 2).
- D5, sandbox root under the worktree's `.lym`: the teardown and leftover paths in `internal/fabricengine/destroy.go`. `RemoveStrayTaskPath` refuses a `.git` (line 1411), and `ignoredTaskPathDeletable` refuses a `.git` at any depth (line 1426). **Breaks** for hubs a halted run never cleared (finding 4).
- D6, facts: `computeRoundFacts` / `reviewFactsRow` (`internal/shedadapters/bouncerfacts.go`) read only reviews and ledgers. D6 states the renderer change. **Survives.**
- D8, retired roles: `agentconfig.Load` → `skillLists` (`internal/agentconfig/config.go`) refuses role keys outside `roles`. The `efforts` check refuses `<role>/<model>` keys for an unknown role. The template lists `treadle-judge` and `treadle-targeting` (`internal/agentconfig/template.yaml:28-29`). **Breaks** (finding 1).
- D9, run settings: `configreg.PrepareRunSettings` (`internal/configreg/runsettings.go`) refuses a module that is not registered. D9's unit loads through configengine, which implies it is registered. **Survives.**
- D10, build engines: `LoomBuildEngines` is also read by `recordOnly` and `onlyRowOf` (`internal/shedbuild/loomcontract.go:27-59`). Adding `HardenFix` keeps the one-build-row rule and the record-only test. **Survives.**

**Findings:**

1. **BLOCKING · design · D8 (decision-record.md, D8, third paragraph) — D8's claim that `internal/agentconfig` refuses no unknown role is false.**
   - Evidence: `Load`'s doc says "It refuses a role key outside the roles lym starts". `skillLists` returns `unusable(... "names %q, which is no role lym starts")` (`internal/agentconfig/config.go`, after line 137). The `<role>/<model>` effort keys are refused the same way. The table test pins both: `UnknownRole` and `PairWithUnknownRole` (`internal/agentconfig/config_test.go:89,97`).
   - Effect: once `treadle-judge` and `treadle-targeting` leave `roles` (`config.go:44-45`, `template.yaml:28-29`), any hub `agents.yaml` layer that names either role fails to load.
   - The criterion "a hub `agents.yaml` naming `treadle-judge` still loads" fails under the change as designed.
   - Fix: ship a `configreg.Reinterpretation` that `hubreconcile` applies to strip the retired keys (PATTERN-config-layers), or keep the retired names accepted on load. Restate D8 and its criterion to match.

2. **MEDIUM · design · D5 (decision-record.md, D5, third paragraph) — `selfreport: false` in the sandbox hub's `loom.yaml` does nothing.**
   - Evidence: the key is retired and ignored on load (`internal/loomengine/config_test.go:212`, case "retired selfreport key ignored").
   - The automatic filing path is the Tier 2 friction reflection, which files through `lym selfreport create` (`internal/friction/doc.go:49`). An empty `friction` turns it off (`internal/loomengine/template.yaml:37`).
   - Fix: drop `selfreport: false` from D5 and its criterion, and rest the override on `friction: ""` alone.

3. **MEDIUM · design · D3 (decision-record.md, D3, Harden perch paragraph) — the lightweight wiring path still reads the Harden-Review round cap from loom config.**
   - Evidence: `loomCLI.reviewBudget` (`internal/loomcli/arm.go`, after `verifySource`) re-loads only `loom.yaml` and `tapestry.yaml`. It returns `loomSegmentBudgets(cfg)`, which reads `cfg.Caps(perch)` from loom config for every entry of `loomSegmentPerches` (`internal/loomcli/wiring.go:1422-1427`).
   - Effect: status and the routing projection would report a cap that is not `hardener.yaml`'s `max_bounces`, or none. The record is silent about this path.
   - Fix: D3 or D9 states that the segment-budget projection, the lightweight reader included, takes the Harden-Review cap from `hardener.yaml`.

4. **LOW · design · D5 (decision-record.md, D5, second paragraph) — "the root goes with the worktree at teardown" has no pin, and fabric's leftover cleaners refuse what the root holds.**
   - Evidence: `RemoveStrayTaskPath` refuses a directory holding a `.git` (`internal/fabricengine/destroy.go:1411`). `ignoredTaskPathDeletable` refuses one with a `.git` at any depth (`destroy.go:1426`).
   - Sandbox hubs are full repositories. D7's halt bound and a crash leave them under the root until a later row's reap.
   - Effect: a run torn down before that reap can leave a task path that neither cleaner deletes, and a later pair of the slug then meets it.
   - Fix: D5 states what teardown does with a root that still holds hubs, with the pin, or has Finalize and the halt path clear the root first.

**Tool calls:**
1. Grepped `treadle` and `roles` in `internal/agentconfig/config.go` and `template.yaml`, to confirm the retired roles sit in the closed role list; my context had only `Load`, `skillLists` and the test.
2. Grepped `.git` and `.lym` handling in `internal/fabricengine/destroy.go`: teardown's treatment of nested repositories was not in my context.
3. Read `destroy.go:1380-1432`, to see which functions refuse a nested `.git` and on which paths.
