```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change, I listed what reads or asserts it outside the diff and checked whether the diff leaves it correct.

**Per change, the set I found and whether it is covered**
- **`checkTaskSetting` takes `unattended`:** `LoadConfig` (`lym config beam --set` through `configreg` `CheckLoad`), `CheckLocalValues` (configreg `CheckValues` for `run_settings`, `internal/configreg/configreg.go:151`) and `LookupWithSource` (lookup, BeamModes). All are covered. `configreg/runsettings_test.go:96-103` still holds for booleans and `yes`.
- **Refusal text is now "must be one of [false true unattended]" plus `<false|true|unattended>`:** the old `TestLoadConfig` row moved into the new dobbyengine test, and `hubgeom` `autonomy_integration_test` asserts `lym config beam --set autonomy=`. Covered.
- **`LookupAutonomy` returns `AutonomyLevel`:** callers are `hubgeom/autonomy.go:58` and `tasksetting_test.go` `lookupAsText` via `On()`. Covered.
- **`hubgeom.Autonomy` returns the level:** callers are `slackcli/wiring.go:131` (`.On()`), `loomcli/driftwiring.go:102` and `pawlwiring.go:154` (`.On()`), `orchcli/paths.go:32` (mapped) and `orchcli/notify.go:101` (stamp). Downstream of those: `slackcli/hook.go:38,84,167` and `doc.go:21` say "autonomy is on", which is right under `On()`. `driftengine/config.go:112` and `loomengine/config.go:575` `ApplyAutonomy(bool)` get `On()`. Covered.
- **Stamp text `; autonomy: <level>`:**
  - Go producers: `NotifyParentHalt` and `NotifyRunFromHalt` (battencli `notices.go`, `wire.go`, `drive.go`, `readopt.go`, `cancelcmd.go`; loomcli `drivewiring.go:161`, `prepush.go:82`) and the dobby. No Go reader parses the stamp. Covered.
  - Prose readers: `contracts/orchrole/orch-role.md:36` and `role-autonomy.md:5,12` key on `autonomy: true`. Not covered; see F1.
- **Board beam modes value is off, on or unattended:** `boardengine/beammodes.go` `settingText` and `modeHeader` pass the value through, and `boardengine/doc.go:157-159` is level-neutral. Covered.
- **Orch prompt line for unattended:** it points at `role-autonomy.md`, and `heddleengine/glossary.md:27` defines autonomy as one mode. Not covered; see F1 and F2.
- **Template comment on `beam.autonomy`:** no other reader. Covered.
- **`orchengine.Paths.Autonomy` signature:** tests in `prompt_test`, `resumecontext_test`, `watcher_test` and `orchcli/start_test` are updated. Covered.
- **Dobby refusal post line (new message):** checked against PATTERN-config-key-names; see F3.
- **PATTERN files:** none are declared changed, and none needs a change. PATTERN-config-layers (`pattern/PATTERN-config-layers.md:125`) names the key without its values. Covered.
- **Commit messages:** no `Pattern-Departure`, `Loosened-Check` or `Root-Cause` trailer is needed. Covered.

**Findings**

- **F1 — MEDIUM, class `decision`, `internal/orchengine/prompt.go:57`.** The unattended prompt line says "as role-autonomy.md beside your role says", but that file holds no unattended rules. `orch-role.md:36` and `role-autonomy.md:5` load the autonomy rules only for a notice stamped `autonomy: true`, so an orch that gets an `autonomy: unattended` stamp loads no autonomy text. The role texts fall outside this part's Takes, and the change description admits the gap. The parent should rule whether the pointer stays or the line drops it until the role-text change lands.
  - Evidence: `contracts/orchrole/role-autonomy.md:5`, `contracts/orchrole/orch-role.md:36`, `internal/orchengine/prompt.go:57`.
- **F2 — LOW, class `consistency`, `internal/heddleengine/glossary.md:27`.** The glossary still defines autonomy as the one mode that reaches the operator for a merge-ready beam or a split approval. The diff now gives unattended as a second level with different reach. This is part of the same deferral as F1.
  - Evidence: `internal/heddleengine/glossary.md:27`.
- **F3 — BLOCKING, class `design`, `internal/beamcli/dobbydeps.go:171`.** The new refusal post names the key informally ("the autonomy setting of run %s ..."), never in full through `configengine`'s key-name helper. PATTERN-config-key-names and the record's constraint require every message to name `beam.autonomy` in full through that helper. The appended refusal does name the key, but the post's own wording does not.
  - Evidence: `internal/beamcli/dobbydeps.go:171`, and the record's Constraints line on PATTERN-config-key-names.

Outside my lens: under PATTERN-config-layers, an older binary's `hubreconcile` walk renames a value no layer admits in the board's `run_settings`. If the board holds `autonomy: unattended`, that walk may rewrite it. This bears on the summary's "two-way" Merge-danger call, which is the handler's to judge.

Tool calls:
1. Grep of non-test Go and of docs, contracts and pattern for two-value autonomy descriptions: the manifest held no repository-wide search of comments and docs.
2. Grep of package docs, specs, stencils and `cmd/lym` for autonomy: the manifest lacked package doc and spec coverage.
