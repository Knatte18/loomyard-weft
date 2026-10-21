# Card 1 — loom-tuned-defaults

**Edit:**
- `internal/loomengine/template.yaml`
- `internal/loomengine#templateConfig`
- `internal/configreg#TestPrepareRunSettings`

**Intent:**
D1's loom values become the embedded template's values, so a hub with no `loom.yaml` unit for a key resolves the tuned value.
In `internal/loomengine/template.yaml` set:
`discussion_lenses: twill-discussion`, `plan_lenses: twill-plan`;
`discussion_reviewer_model: ["opus[medium]", "opus[medium]", "sonnet[high]"]`, `plan_reviewer_model: ["opus[medium]", "sonnet[high]"]`;
`webster_reviewer_model`, `webster_fixer_model`, `merge_reviewer_model` and `merge_fixer_model` to `sonnet[medium]`;
`discussion_reviewer_fan: ["review-discussion", ""]`, `plan_reviewer_fan: ["review-plan", ""]`, `webster_reviewer_fan` and `merge_reviewer_fan` to `["review-code", ""]`;
`discussion_fixer_fan: fix-discussion`, `plan_fixer_fan: fix-plan`, `webster_fixer_fan` and `merge_fixer_fan` to `fix-code`;
`driver: haiku[medium]`, `discussion_max_bounces: 2`, `webster_max_bounces: 1`, `webster_auto_accept: true`.
Each changed key's line comment is rewritten to describe the new default instead of the old one: an empty value is no longer "the default", `webster_auto_accept`'s comment says the template turns the accept on and `false` restores the soft-cap rule, and `driver`'s comment says the llm driver runs only when a hub, a module's driver override, a task's run settings or `--driver` selects `llm` and a hub pins the model back with `lyx config loom --layer default --set driver=sonnet[medium]`.
Every fan and lens named exists in `internal/lenses/template.yaml` today (`twill-discussion`, `twill-plan`, `review-discussion`, `review-plan`, `review-code`, `fix-discussion`, `fix-plan`, `fix-code`), so no lenses change is needed.

What the new defaults let through, as D1 records: Webster-Review gets one judged round and accepts a harvested round at the spent budget with its open findings, BLOCKING ones included, Discussion-Review two rounds, and the Webster-, Merge-Review reviewers and fixers and the later Plan- and Discussion-Review reviewers run lighter models.
The bound is D1's: every value is a loom key a hub pins back with `lyx config loom --layer default --set`, the verify gates and PR-Gate still check the code, and an existing hub reaches the values only through the walk's one-time strip (card 8).

`templateConfig`, the `Config` the template resolves to in `loomengine`'s tests, takes the new values, so every test comparing a load with it pins the new defaults.
`TestPrepareRunSettings`'s row whose `wantDocs` spells `webster_reviewer_model: sonnet[high]` and `merge_reviewer_model: sonnet[high]` with their template line comments follows the template's new values and comments, so the row still pins that a run-settings document carries the template's line comment.
The implementer greps the test files for the old loom defaults (`sonnet[high]` as a Webster or Merge reviewer default, `discussion_max_bounces: ~`, `webster_auto_accept: false`, `driver: sonnet[medium]`, an empty `*_fan` or `*_lenses` taken as the default) and fixes, in this card's commit, any further assertion that reads a template default rather than a value it seeds itself.
The test files that load `loomengine.ConfigTemplate()` or wire a loom recipe from it are the ones to check first: `internal/loomcli`'s `wiring_test.go`, `specfill_test.go`, `tacklerfor_test.go`, `smoke_helpers_test.go`, `smoke_driverstrand_test.go`, `smoke_starttail_test.go`, `integration_driverbootstrap_test.go` and `circling_test.go`, and `internal/loomrecipe`'s `reviewbudget_test.go`, `fansettings_test.go` and `escalation_test.go`; a member found reading a changed default joins this card's commit.

**ImpactSummary:** the loom template's defaults and comments; two test fixtures that mirror the template follow it, and every loader of loom config resolves the new values.
