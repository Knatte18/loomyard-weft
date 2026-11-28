```yaml
lens: code-scope
outcome: reported
```

Task: map every change in the diff to the amended board entry's goal points and Added-after-Discussion entries (lens code-scope).

**Per change: the goal point it traces to**
1. `beamsetting.go`: adds the constants `AutonomyOff`, `AutonomyOn` and `AutonomyUnattended`, adds `AutonomyValues()`, and rewords the `AutonomyKey` comment. This is forced by change 2, because the values are declared once beside the other per-task values (002 diff:19-34).
2. `dobbyengine/config.go` `checkTaskSetting`: accepts `unattended`, and refuses anything else naming the three values and `lym config beam --set`. Traces to Delivers "three levels on every path that checks the key" and "a refusal naming the three values and `lym config beam`" (004 diff:62-72). The `LoadConfig` and `CheckLocalValues` comments are forced by this change.
3. `dobbyengine/config.go` adds the `AutonomyLevel` string type, its constants and `On()`. `tasksetting.go` makes `LookupAutonomy` return the level and read a refusal as `AutonomyOff`. Traces to Exports and to Delivers "a refused value read as `false`" (004 diff:23-42, 193-202).
4. `hubgeom/autonomy.go`: `Autonomy` returns the level, and `AutonomyWithRefusal` is new. Traces to Exports ("`hubgeom.Autonomy` returns it"). `AutonomyWithRefusal` serves Delivers "posted once per episode to the orch" (005 diff:31-75).
5. `beamcli/dobbydeps.go` `runAutonomyStamps`, and the dobby's three Post seams switched over to it. Traces to Delivers "posted once per episode to the orch" and "the level on the `autonomy:` stamp" (001 diff:9-50, 59-84).
6. `orchcli/notify.go`: `StampAutonomy`, with `WithRunAutonomy` rerouted through it. Traces to Delivers "the level on the `autonomy:` stamp" (007 diff:17-30).
7. `hubgeom/beammodes.go` `autonomyTexts`, and the comment in `boardengine/beammodes.go`. Traces to Delivers "`off`, `on` or `unattended` in the beam modes" (005 diff:187-212; 003 diff:10).
8. `orchengine/prompt.go`: `autonomyUnattendedLine` and a switch over the level. `orchengine/state.go` adds its own `AutonomyLevel` int type, and `orchcli/paths.go` adds `orchAutonomy`. Traces to Delivers "the level named in the orch prompt" (008 diff:9-39, 110-122; 007 diff:58-74). Commit 3acab4e7 explains why orchengine has its own type: it keeps orchengine within its import allowlist. That separate type is an edit the prompt change forces, and Exports does not forbid it, since Exports fixes only dobbyengine's type.
9. `slackcli/wiring.go`, `loomcli/driftwiring.go` and `loomcli/pawlwiring.go` read the level through `.On()`. Traces to Delivers "Every reader that asks only whether autonomy is on reads `true` and `unattended` as on" (009 diff:10; 006 diff:12, 25).
10. `template.yaml`: the `beam.autonomy` comment now names the three values and the bound on older builds. Traces to Delivers "the level named in … the template comment" (004 diff:250).
11. The `doc.go` updates in `beamcli`, `dobbyengine` and `hubgeom`, and every test file change, are forced by changes 2 through 9.

**Per goal point: the changes that build it**
- Takes, D1 whole (`LookupAutonomy`, `checkTaskSetting` and its callers, the template comment, `hubgeom.Autonomy` and its callers in slackcli, loomcli, orchcli and beamcli, the beam modes, the orch prompt line): changes 2 through 10.
- Delivers, three levels on every checking path: change 2. The lookup, `LoadConfig`, `CheckLocalValues` and `lym config beam --set` (through `CheckLoad`, which runs `LoadConfig`) all share it.
- Delivers, a refusal naming the three values and `lym config beam`: change 2.
- Delivers, a refused value read as `false` and posted once per episode: changes 3, 4 and 5.
- Delivers, the level on the `autonomy:` stamp: changes 5 and 6.
- Delivers, `off`, `on` or `unattended` in the beam modes: change 7.
- Delivers, the level in the orch prompt and the template comment: changes 8 and 10.
- Delivers, readers that ask only on/off read `true` and `unattended` as on: change 9.
- Exports, `dobbyengine.AutonomyLevel` with its constants and `On()`, returned by `LookupAutonomy` and `hubgeom.Autonomy`: changes 3 and 4.
- Added after Discussion (parent, F37), on stop identity and stop records: no change builds it. It amends D3 and D7, and the entry's Takes names D1 only, so it is outside this part's scope.
- Added after Discussion (arbiter, F37): no change builds it, for the same reason. It records only that D3 and D7 stand as written.

**Findings**
None. Every change traces to a goal point or is forced by one. Every goal point this part takes is built by at least one change. The two Added-after-Discussion entries amend decisions outside this part's Takes.

Out of lens: `contracts/orchrole/role-autonomy.md:5` and `orch-role.md:36` still key on `autonomy: true`. That belongs to the affected-set lens and the handler.

Tool calls: none.
