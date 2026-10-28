```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. Proposals the fixer carried out, each challenged for a case where it is wrong:

- **F1(1)**, dropping the local `nosuchfan` subtest: holds. `ValidateFanKeys` runs on the merged config (`internal/loomengine/config.go:1225`). Row 610 pins the refusal text. The new hub-stack row pins that a local value reaches the merged config. Nothing else in card 1 refers to the dropped subtest.
- **F1(2)**, removing the `TestModules_LocalLayerRefusals` Edit and its row: holds. The test has no `code_reviewer_fan` row to update, and its doc limits it to keys no loader pins (`internal/configreg/configreg_test.go:180`). Card 1's ImpactSummary still says "local-layer … rows change or join". That stays true through `TestLoadConfig_Refuses`' loop change, so nothing breaks.
- **F1(3)**, dropping the three refused run-settings rows: holds. `PrepareRunSettings` refuses through `checkLocalDocument` (`internal/configreg/runsettings.go`, around line 83). That reads the same `LocalDenied` set `wantLocalDenied` pins, so the five keys staying refused is still pinned.
- **F2**, the darn/pawl sentence: one possible case, below.
- **F3**, the header exception: holds. The wording matches the `reviewerOnly` set card 4 defines.
- **F4**, the recovery: holds. `--set` is repeatable and runs one `CheckLoad` per call (`internal/configcli/doc.go:49`, `:66`), so fixing every fan in one call passes the check.

Findings:

- **LOW**, `_lym/plan/01-code-reviewer-fan-local.md` Intent, the added sentence "a darn or pawl pair reads `loom.code_reviewer_fan` through `configreg.SettingLookup`, which resolves no fan…". The split between pair kinds that the proposal states may not hold.
  - Evidence: the shed `Setting` lookup is wired at `internal/loomcli/wiring.go:761` and `internal/battencli/wire.go:1444`/`1447`. The same loomcli wiring file calls `loomengine.LoadConfig` at line 378.
  - No production Go file outside `internal/loomengine` and `internal/configreg/keymove.go` names `code_reviewer_fan`. So a darn or pawl pair reads the key through a recipe, run either by loomcli's wiring or by batten's.
  - If loomcli's wiring runs that recipe, `LoadConfig` at line 378 may refuse the wrong name before the Code-Review round. The sentence's "there the wrong name is refused when the Code-Review round resolves the recorded fan" would then be wrong.
  - I did not confirm which wiring a darn or pawl pair uses. The fixer should check whether a darn or pawl run goes through `internal/loomcli/wiring.go:378` and, if it does, narrow the sentence to the route that skips `LoadConfig`.

Tool calls:

1. Grepped production uses of `code_reviewer_fan`, `wantLocalDenied` and `CategoryGuard`. My context did not show how darn/pawl read the key or how run settings refuse.
2. Grepped `runsettings.go` and the key's readers. My context lacked the run-settings refusal mechanism.
3. Located `PrepareRunSettings` and the `LocalDenied` owners. That checks whether F1(3)'s claim rests on the same set.
4. Grepped `runsettings.go` for the refusal call, `checkLocalDocument`. That confirms the run-settings refusal uses `LocalDenied`.
5. Grepped `SettingLookup` callers. That checks F2's claim about the darn/pawl read route.
