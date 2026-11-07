---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/claudeengine/command.go:443
    summary: The explicit effort an opened session gets from agents.yaml in OpenLaunch has no test.
    fix: Add a row to the existing OpenLaunch test table in internal/shuttleengine/claudeengine/command_test.go (or subagents_test.go) that passes an OpenSpec with no Effort and a cfg.Agents from shuttlefake.Agents, and asserts the launch line carries --effort with the default entry, plus a row with a full Claude model id asserting the model alias entry wins (claude-haiku-5-5 gives the haiku entry).
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/shuttleengine/config.go:167
    summary: LoadConfig loading agents.yaml into Config.Agents, and failing the call on an unusable file, has no test.
    fix: Add a case to the LoadConfig tests in internal/shuttleengine (config_test.go) that loads with a hub agents.yaml missing the default effort and asserts LoadConfig returns an error wrapping agentconfig.ErrUnusable, and a case with no agents.yaml asserting Agents resolves a skill list for a role.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/orchcli/cli.go:204
    summary: The orch reload list resolved from agents.yaml in wireCLI and handed to the watcher has no test; tests set orchSkills by hand.
    fix: Add an orchcli test that runs wireCLI (or the verb that builds the watcher) over a hub whose agents.yaml sets a distinct orch list and asserts the watcher receives that list, not the template's.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: design
    location: internal/shedadapters/bouncer.go:1732
    summary: No test asserts ConfigRole for the bouncer seed and judge, the discussion advisor or board-tidy, so a wrong role there goes unseen.
    fix: Extend the existing spec assertions in internal/shedadapters/bouncer_judge_test.go, internal/loomengine/discussiontable_test.go and internal/dobbyengine/tidy_test.go to assert spec.ConfigRole equals agentconfig.RoleBouncerSeed, RoleBouncerJudge, RoleDiscussionAdvisor and RoleBoardTidy respectively.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/agentconfig/config.go:197
    summary: checkEfforts accepts any non-empty effort value, so a typo in a model or role entry reaches every Agent call through the passage and fails only there.
    fix: In checkEfforts, refuse a value outside Claude Code's effort vocabulary (low, medium, high, xhigh, max) with the marked content failure naming the key, by taking the vocabulary from one declaration shared with claudeengine's validEfforts or by declaring it in agentconfig and having claudeengine read it; add a row to TestLoad_RefusesUnusableContent.
    origin: lens:code-correctness
  - id: F6
    severity: LOW
    class: scope
    location: internal/configcli/configcli.go:479
    summary: The diff deletes the config help line "A value the loader refuses is refused, naming the key.", which the goal does not ask for.
    fix: Restore the deleted line in buildConfigLong and revert the matching one-line change in internal/configcli/configcli_test.go, unless a gate rule on help length forced it, in which case name that rule in the commit message.
    origin: lens:code-affected-set
---

### [MEDIUM] OpenLaunch effort is untested

**Location:** internal/shuttleengine/claudeengine/command.go:443
**Issue:** The goal asks every session lym launches, opened sessions included, to get an explicit effort.
`OpenLaunch` now calls `effortFor` with no role, and no test fails if that call is removed or its model alias breaks.
The `claude-haiku-5-5` to `haiku` match the change description claims is exercised nowhere in the tests the diff adds.

### [LOW] LoadConfig's agents.yaml load is untested

**Location:** internal/shuttleengine/config.go:167
**Issue:** `LoadConfig` now fails when `agents.yaml` is unusable and fills `Config.Agents`.
`internal/agentconfig/config_test.go` pins `agentconfig.Load`, but nothing pins that shuttle's config load calls it and returns its error.

### [LOW] Orch reload list wiring is untested

**Location:** internal/orchcli/cli.go:204
**Issue:** `wireCLI` resolves the orch list and `watch.go` passes it to the watcher, replacing a package variable.
`cli_test.go` only sets `orchSkills` by hand, so a wiring that sends the wrong list passes.

### [LOW] Four roles' ConfigRole is not asserted

**Location:** internal/shedadapters/bouncer.go:1732
**Issue:** The bouncer seed and judge, the discussion advisor and board-tidy name their `agents.yaml` role in `ConfigRole`, and no test reads it back.
A wrong constant would start the session with another role's skills.

### [LOW] Effort values are not validated at load

**Location:** internal/agentconfig/config.go:197
**Issue:** `checkEfforts` refuses an empty effort but not an unknown one.
`subagentEffortEnv` validates only the role-level fallback and `Prepare` only the launch effort, while `subagentPassage` tells every session to put each model's value in its Agent calls.
A typo in a model entry therefore surfaces as a failing Agent call.

### [LOW] Unlicensed removal of a config help line

**Location:** internal/configcli/configcli.go:479
**Issue:** The diff removes the help sentence about a refused value being refused naming the key.
The goal does not touch `lym config` help, and the commit message "give shuttle test runners the agents config, mark its seam, and trim config help" gives no reason tied to it.

## Rejected

- modelAlias mishandles legacy ids such as `claude-3-5-sonnet-20241022` and bracketed aliases such as `opus[1m]` (lens:code-correctness): the models lym launches use the current `claude-<alias>-<n>-<n>` ids and plain aliases, and the fork itself could not show a legacy or bracketed id in use.
- The backstop rides lym's per-session settings `env` instead of heddle's `.claude/settings.local.json` (handler): the goal asks for the backstop "where it is a setting", the value differs per role, which heddle's static entries cannot carry, and the change description states the departure; the unverified precedence of `CLAUDE_CODE_SUBAGENT_EFFORT` against an explicit Agent `effort` is the writer's own listed hand check.
- Standalone `claude-haiku-5-5` alias row as its own NIT (lens:code-tests): folded into F1.
- Missing roles, skill order and dedup, load refusal, codex isolation, stencil wording, PATTERN and docs changes (all lenses and handler): each was traced and found to match the goal, so there is nothing to record.
