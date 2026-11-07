# Fixer-report, code review round 1

## Changes

- **F1** (fixed as proposed, with one variant): `internal/shuttleengine/claudeengine/command_test.go`.
  `TestOpenLaunch` gained a `withAgents` field and two rows: a spec with no effort takes the `agents.yaml` default, and a full model id (`claude-fable-5-1`) takes its alias entry.
  Variant: the review proposed `claude-haiku-5-5` and a haiku entry, but the shipped template holds no haiku entry.
  The rows use the template's `fable` entry instead, which exercises the same alias path without a custom `agents.yaml`.
- **F2** (fixed as proposed): `internal/shuttleengine/config_test.go` gained `TestLoadConfig_AgentsYAML`.
  Its rows are an absent file resolving the template, a hub file overriding a role's skills, and an unusable file failing the load with `agentconfig.ErrUnusable`.
- **F3** (fixed as proposed, narrowed to `wireCLI`): new `internal/orchcli/agents_integration_test.go`, tier 1, over a hub forged with `hubforge`.
  It writes a distinct orch list into the hub's `agents.yaml` and asserts `wireCLI` leaves exactly that list in `orchSkills`.
  It does not cover `watch.go` passing `c.orchSkills` on, which is a direct field read.
- **F5** (fixed as proposed): `internal/agentconfig/config.go` declares the effort vocabulary once (`Efforts()`), and `checkEfforts` refuses a value outside it, naming the key.
  `claudeengine.validateEffort` reads the same list through `agentconfig.Efforts()`, and its error text is unchanged.
  The `validEfforts` map is removed.
  `template.yaml` now says the values are validated when the file loads.
  `TestLoad_RefusesUnusableContent` gained an `UnknownEffort` row.
- **F6** (fixed with my own change, not restored): the review's proposal was to restore the deleted help line, and I did not keep it.
  The restored line took the `lym config` Long text to 2006 bytes, and `TestCLIForm_EveryCommand` (`cmd/lym/clitree_test.go:413`) caps Long at 2000 bytes.
  That cap is the gate rule that forced the original deletion in `a1568d23`, so `internal/configcli/configcli.go` is back to the trimmed text and the finding is closed by naming the rule.
  The review also asked to revert "the matching one-line change" in `configcli_test.go`.
  That change is the menu test's module index from commit `00a08d6d`, which exists because the agents module shifted the module order; it has nothing to do with the help line, so it stays.
- **Self-check follow-up** (own change, from the fork findings below): `subagentEffortEnv` no longer validates the effort and no longer returns an error, and its caller in `Prepare` drops the error branch.

Commits, one per finding: F1 `2c8dd67e`, F2 `5b574aaf`, F3 `5874ba76`, F5 `f7317f62`, F6 `37bf1136` (since reverted by the second gate repair below), then the F5 follow-up.
Tier 0 passed on `agentconfig`, `shuttleengine`, `claudeengine`, `configcli` and `orchcli`, and tier 1 passed on `orchcli` with the new test running.

- **Gate repair:** the gate refused the two-line comment of `TestLoadConfig_AgentsYAML` as a wrapped break.
  It now holds two sentences, one per line, with no change to the test.
- **Second gate repair:** the verify run failed `TestCLIForm_EveryCommand` because the restored F6 help line pushed the `lym config` Long text to 2006 bytes against the 2000-byte cap.
  I removed the line again, which brings the text back under the cap, and the F6 entry above records the cap as the reason.

## Disputed

- **F4**, ground `premise false`: the four `ConfigRole` assertions the review says are missing already exist.
  - Bouncer seed and judge: `internal/shedadapters/bouncer_judge_test.go:136-138` sets `wantConfigRole` to `agentconfig.RoleBouncerSeed` or `RoleBouncerJudge`, and `:181` asserts `GotSpec.ConfigRole` against it.
  - Discussion advisor: `internal/loomengine/discussiontable_test.go:164` asserts `advisor.ConfigRole != agentconfig.RoleDiscussionAdvisor`.
  - Board-tidy: `internal/dobbyengine/tidy_test.go:344` asserts `spec.ConfigRole != agentconfig.RoleBoardTidy`.

## Deferred

Nothing deferred.

## Self-check

All four forks returned a report, saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-consistency, LOW: `subagentEffortEnv` still validated an effort `Load` now refuses earlier. Fixed with my own change (it removes the dead check rather than documenting it).
- fix-code-correctness: no findings. It noted the same unreachable check outside its lens, which the line above covers.
- fix-proposal: no findings. It made the same observation, covered above.
- fix-affected-set: no findings. It confirmed the F4 dispute against the same four assertions.
