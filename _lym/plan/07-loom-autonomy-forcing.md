# Card 7 — loom-autonomy-forcing

**Create:**
- `plan:internal/loomengine#ApplyAutonomy` -> `func ApplyAutonomy(cfg Config, autonomous bool) Config`
- `plan:internal/loomengine#AutonomyForcedSetting` -> `func AutonomyForcedSetting(key string) (value bool, forced bool)`
- `plan:internal/loomcli#autonomySettingLookup` -> `func autonomySettingLookup(lookup func(module, key string) (any, error), autonomous func() bool) func(module, key string) (any, error)`
- `plan:internal/loomengine#TestApplyAutonomy` -> `func TestApplyAutonomy(t *testing.T)`
- `plan:internal/loomcli#TestAutonomySettingLookup` -> `func TestAutonomySettingLookup(t *testing.T)`
- `plan:internal/loomcli#TestRunAutonomy_ForcesTheRunsKeysOverARealHub` -> `func TestRunAutonomy_ForcesTheRunsKeysOverARealHub(t *testing.T)`

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#pullRequestDepsGuardedBy`
- `docs/overview.md`

**Uses:**
- `plan:internal/loomcli#runAutonomy`
- `internal/loomcli#driftDeps`
- `plan:internal/driftengine#AutonomyForcedSetting`
- `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag`
- `internal/loomengine#DiscussionTable`
- `internal/loomengine#TestDiscussionTable`
- `internal/loomshed#raddleWrite.firstMode`
- `internal/loomshed#TestRaddleWrite`
- `internal/configreg#SettingLookup`

**Intent:** D3 on loom, and D4's new timeout key.

Forcing:

- `ApplyAutonomy` returns `cfg`; with `autonomous` true it sets `PRReviewAskParent` true, `DiscussionInteractive` false and `RaddleAutoApprove` true.
  `code_review_ask_parent` and `discussion_review_ask_parent` keep their own values.
- `AutonomyForcedSetting` answers those three keys forced to the same values, and `forced` false for every other loom key.
- `wire` applies `loomengine.ApplyAutonomy(loomCfg, runAutonomy(location))` right after `withTapestry`,
  before any reader uses a field, so `DiscussionTable` spawns the lead non-interactive with `discussionModeRules`' autonomous text.
- `wire` builds its setting lookup as `autonomySettingLookup(configreg.SettingLookup(...), func() bool { return runAutonomy(location) })`.
  The wrapper answers a forced loom key through `loomengine.AutonomyForcedSetting` and a forced drift key through `driftengine.AutonomyForcedSetting` while `autonomous()` reads true,
  and passes every other call to `lookup`.
  So `raddle_auto_approve` reads `true` under autonomy whatever a profile's raddle block wrote to the local layer.
  `autonomous` is read on each forced key's call, never captured at wiring.
- Raddle-Write records the run's mode in its state on its first call (`raddleWrite.firstMode`) and reads it from there after,
  so a raddle run already past that call keeps its recorded mode; `TestRaddleWrite` already pins the single read.
- `pullRequestDepsGuardedBy`'s `ParentReviewWanted` returns `loomengine.ApplyAutonomy(cfg, runAutonomy(l)).PRReviewAskParent`.

The new key:

- `Config` gains `PRReviewParentTimeoutMin int` (`yaml:"pr_review_parent_timeout_min"`), the minutes from a PR-Review ask to the parent to its deadline.
- `template.yaml` gains `pr_review_parent_timeout_min: 60` with a comment in the style of `code_review_parent_timeout_min`'s:
  a positive integer, 60 by default, settable per run through run_settings.
- `ConfigSpec` classifies it `none`, as its sibling parent timeouts are.
- `LoadConfig` refuses a value below 1 with the sibling keys' message and way forward.
- `docs/overview.md`'s sentence that `discussion_review_parent_timeout_min` sets the PR-Review ask's deadline names `pr_review_parent_timeout_min` instead.

Tests:

- `TestApplyAutonomy` (tier0): true forces the three keys from layered `pr_review_ask_parent: false`, `discussion_interactive: true` and `raddle_auto_approve: false`,
  and keeps `code_review_ask_parent` and `discussion_review_ask_parent` as layered; false keeps all five as layered.
- `TestAutonomySettingLookup` (tier0), over a stub lookup answering `raddle_auto_approve` false as a profile's raddle block leaves it:
  under `autonomous` true it answers `true` for `loom.raddle_auto_approve`, `false` for `drift.judge_interactive`, and the stub's value for `loom.raddle`;
  under false it answers the stub's values; a stub error passes through for an unforced key.
- `TestRunAutonomy_ForcesTheRunsKeysOverARealHub` (tier1), over a `hubforge.CopyHub` hub with a beam pair and a task pair added from it,
  whose loom and drift layers hold `pr_review_ask_parent: false`, `discussion_interactive: true`, `judge_ask_parent: false` and `judge_interactive: true`,
  and whose local layer holds `raddle_auto_approve: false` as a profile's raddle block leaves it.
  With the beam's own beam.yaml setting `autonomy: true` and the task setting nothing, D3's "parent worktree autonomous":
  - `pullRequestDepsGuardedBy(...).ParentReviewWanted()` reads true;
  - `driftDeps(...).Config` reads `JudgeAskParent` true, `JudgeInteractive` false and `Autonomous` true;
  - `wire` over the task builds an env whose `DiscussionTable()` lead seat is non-interactive with the autonomous mode text,
    and whose `Setting("loom", "raddle_auto_approve")` reads true, so a raddle proposal files with no approval.
  With the beam's file set `false`, each reads its layered value.
- `TestLoadConfig_Loads` pins `pr_review_parent_timeout_min` at its default 60 and at a layered value;
  `TestLoadConfig_Refuses` gains rows for `0` and `-1`, asserting "must be at least 1" and the positive-integer way forward.

**ImpactSummary:** Two loomcli config readers and the run's setting lookup take the forced values; loom's spec and template gain one key, and every other loom config reader and key is unchanged.
