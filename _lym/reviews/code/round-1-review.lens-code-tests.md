```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (test that pins it, or none)**

1. Config role list at load, refusal of an unknown, null or malformed role, and refusal of a missing default effort: pinned by `internal/agentconfig/config_test.go` (`TestLoad_RefusesUnusableContent`, `TestLoad_TemplateListsEveryRole`).
2. Hub-file fallback to the template's list: pinned by `config_test.go` (`HubFile_OtherRoleKeepsTemplate`).
3. Role list ahead of extra skills, repeats dropped: pinned by `config_test.go` (`TestConfig_SkillsAppendsExtraWithoutRepeats`) and `internal/shuttleengine/configrole_test.go` (`RoleListLeadsTheSpecSkills`).
4. Refusal of a role with no list at `Start`: pinned by `configrole_test.go` (`RoleWithoutListIsRefused`).
5. Effort precedence (`role/model`, role, model, default): pinned by `config_test.go` (`TestConfig_EffortTakesTheMostSpecificKey`).
6. `ModelEfforts`: pinned by `config_test.go` (`TestConfig_ModelEffortsListsTheModelsTheRoleMeets`).
7. Effort applied in `Prepare`, including the `claude-fable-5-1` to `fable` alias and an explicit spec effort winning: pinned by `internal/shuttleengine/claudeengine/subagents_test.go` (`TestPrepare_EffortFromAgentsConfig`). The row list shows only `fable`, so `claude-haiku-5-5` to `haiku` is not visibly covered.
8. Appended subagent passage and the `CLAUDE_CODE_SUBAGENT_EFFORT` env in the settings: pinned by `subagents_test.go` (`TestPrepare_TellsTheSessionWhatItsSubagentsGet`).
9. No agents config means no passage and no env: pinned by `subagents_test.go` (`TestPrepare_NoAgentsConfigWritesNoSubagentRule`).
10. Each spawn site passes the right `ConfigRole` (tackler, treadle, sampler, webster, darn, merge fixer, describer, conflict resolver, PR reviewer, drift, gate fixer, plan, discussion, orch, arbiter, friction, burler, verify fixer): pinned by per-site asserts in their `_test.go` files.
11. The `agentconfig` leaf import set: pinned by `internal/agentconfig/leaf_enforcement_test.go`.
12. `OpenLaunch` effort for an opened session: none. `command_test.go` has no `effortFor` or agents row. The only effort tests of opened sessions are the existing ones.
13. `LoadConfig` loading `Agents` and failing when `agents.yaml` is invalid: none. No `shuttleengine/config_test.go` row appears in the diff.
14. Orch reload re-sending the `agents.yaml` list: `wireCLI` resolves `orchSkills` and `watch.go` passes it to the watcher. No test reaches `wireCLI`. `internal/orchcli/cli_test.go` only hand-sets `orchSkills`.
15. Gate-fixer role move: `internal/gatefixer/spawn_test.go` covers `SpawnOrAttach`, and `internal/websterengine/verifyfixer_test.go` covers the verify fixer. `internal/gatecli` and `internal/loomcli/gatefixer.go` have no diff in their tests, but both go through `gatefixer.SpawnOrAttach`, so the role is set there.
16. Bouncer seed and judge, discussion advisor, board-tidy, treadle-targeting, fix-review and raddle roles: the grep shows no `ConfigRole` assert for bouncer seed, discussion advisor or board-tidy. Treadle targeting (`judge_test.go:98`), fix-review (`fixreview.go:170`, not in the grep) and raddle (`raddletable_test.go:68`) are partly or ambiguously covered, so I could not confirm them.
17. Codex or non-Claude launches getting no Claude effort: none. No test pins this for `codexengine`. The code path keeps effort inside `claudeengine`, so the risk is low.

**Findings**

- MEDIUM, design: `OpenLaunch` effort has no test (`internal/shuttleengine/claudeengine/command.go:443`). The explicit `--effort` for an opened session is a stated goal item. `effortFor(cfg.Agents, "", ...)` could be broken with no test failing.
- MEDIUM, design: `LoadConfig` loading `agents.yaml` into `Config.Agents` and failing the call on a bad file has no test (`internal/shuttleengine/config.go:167`). Without it, "refusal at load" has no pin at the shuttle level.
- LOW, design: the orch reload list from `agents.yaml` has no test (`internal/orchcli/cli.go:204`, `internal/orchcli/watch.go:132`). The `orchSkills` wiring in `wireCLI` is unexercised, and tests only set the field by hand.
- LOW, design: `ConfigRole` for the bouncer seed and judge (`internal/shedadapters/bouncer.go:1732` and `:1886`), the discussion advisor (`internal/loomengine/discussiontable.go:148`) and board-tidy (`internal/dobbyengine/tidy.go:92`) shows no assert. The first test calls were the grep and the stat in the second, and both showed no match for these roles. A wrong or missing `ConfigRole` there would start a role with no list, which `Start` would only catch if the spec reaches shuttle.
- NIT, design: the `claude-haiku-5-5` to `haiku` alias match is untested. The listed `FullModelID` row covers only `claude-fable-5-1`, so `modelAlias` is pinned for one id only.

**Tool calls (2 of 6)**
1. `grep` over the new test files, the `OpenLaunch` and `CLAUDE_CODE_SUBAGENT_EFFORT` tests, the orch tests and every `ConfigRole` assert in tests: my context held no test files.
2. `git diff --stat` plus `grep` for `effortFor` and agents in `command_test.go` and the shuttle config test, and for role constants in tests: checked whether the opened-session, `LoadConfig` and remaining-role tests exist.
