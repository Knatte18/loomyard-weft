# overview-seats: single LLM producer removed, seats named, loom config axes renamed

All six batches of the plan report `done`.
This session ran the last two; the earlier four (01 shed-bounce-routed-to, 07 checklist-pause-net-time, 11 discussion-write-seats, 12 plan-write-seats) were already done when it resumed.

## Batch 17: single-llm-delete (head `3ad0cb42904db86fcdefa0ba928c5630186912af`)

- Cards 17 and 18 landed.
- `SingleLLMProducer`, `SpecSource` and the `SingleLLM` registry row are deleted, along with `entries_singlellm.go` and the two `singlellm` test files.
- `shuttle.go` now declares only the `Shuttle` seam.
- The spec builders `DiscussionSpec`, `PlanSpec`, `ReworkSpec`, `DescribeSpec`, `DarnDescribeSpec` and `DarnSpec` are deleted with their composers.
- `DescribeInputs`, `DarnDescribeInputs` and `DarnInputs` lost `ParentName`.
- Tests: `TestMultiLLMProducer_MapsTheLeadResult` replaces the old comparison test, and the smoke-lead tests run through a one-seat `MultiLLMProducer` helper.
- Deviations from the cards, as the fork reported them: the advisor stencil constant in `shedbuild` was renamed to `advisorStencil`, and stale import-allowlist entries were removed from two seam tests.
- Scope guard deviations (informational): `cmd/lyx/pathrules_test.go`, `internal/darnengine/spec.go`, `internal/landingshed/describe.go`, `internal/landingshed/describe_test.go`, `internal/landingshed/seam_enforcement_test.go`, `internal/loomcli/smoke_attachprobe_test.go`, `internal/loomengine/rework.go`, `internal/mergeresolve/spec.go`, `internal/shedadapters/singlellm_test.go`, `internal/shedbuild/build_engines_test.go`, `internal/shedrecipe/entries_singlellm.go`, `internal/shedrecipe/entries_singlellm_test.go` and `internal/shedrecipe/seam_enforcement_test.go`.
- `cmd/lyx/pathrules_test.go` still cites a test that no longer exists, `TestPlanSpec_PatternDirective`; the fork reworded only the `PlanSpec` case reference on that line.

## Batch 19: loom-config-axes (head `5ce016e8913a852b06701f3c3ee869830614312b`)

- Cards 19 and 20 landed.
- `discussion_producer` and `plan_producer` are retired.
- `discussion_advisors` (named entries), `discussion_lenses` and `plan_lenses` replace them as guard keys.
- `DiscussionTable` and `PlanTable` build from the new keys.
- `MigrateConfig` rewrites old files and has its own test.
- `RoleChair`, `RoleAdvisor` and `AdvisorName` are deleted from `internal/seatengine/table.go`.
- Beyond the cards: one line in `contracts/specs/shed-recipe-spec.md` was reworded, the `plan_lenses` comments in `entries_planwrite.go` were updated, two stale messages in `internal/loomcli/wiring_test.go` were fixed, and the `seats` rows were dropped from `TestNewParentReviewConfig_KeepAliveInterval`.
- Scope guard deviations (informational): `contracts/specs/shed-recipe-spec.md`, `internal/loomcli/wiring_test.go`, `internal/loomengine/config.go`, `internal/loomengine/config_test.go`, `internal/loomengine/discussiontable_test.go`, `internal/loomengine/plan_test.go` and `internal/shedrecipe/entries_planwrite.go`.

Neither batch produced warnings.
