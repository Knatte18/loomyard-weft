---
round: 1
focus:
  - "internal/shuttleengine/run.go and spec.go: the parent added `Provider: engineName` to the run record and the recordless Send/Interrupt path; the task embeds `ModelChoice` in `Spec` and records `Result` engine. Settle whether the two record the same engine name for the same run, and whether `engineFor(state)` and the recordless path pick the engine the task's Webster audit and `Spec.Requires` checks expect."
  - "Engine-owned config (`claudeengine/config.go`, `command.go`, `prepare_test.go`, `command_test.go`) against the task's `Spec.Requires` and `ModelChoice` handling: settle whether any task code or test still reads the removed shuttle-config `claude_deny_*` keys or assumes the old `Spec`/`Model`/`Effort`/`Engine` field layout."
  - "`PATTERN-model-choice-mapping` (task) against the parent's new code: settle whether any production code the parent added (creel stage 1, `shuttleengines`, `creelfake`, `loomcli/wiring.go`, `webstercli/wiring.go`) reads a `modelspec.Resolved` field by hand, and whether the scan test still passes the pattern's rule on the merged tree."
  - "`PATTERN-shuttle-provider-seam` and `PATTERN-leaf-packages` changes from the parent against the task's `ModelChoice` embedding in `shuttleengine`: settle whether the task's types or fields put a provider name or a forbidden import into a package those entries guard."
  - "Overlapping loom files (`loomengine/config.go`, `plan.go`, `discussiontable.go`, and their tests) and `loomcli/wiring.go`: settle whether the parent's edits still compile-by-meaning with the task's resolvers returning `shuttleengine.ModelChoice`, for example a renamed or re-signatured function one side still calls with the other side's shape."
  - "`internal/testkit/envkit/envkit.go` and the overlapping test files (`enginedispatch_test.go`, `smoke_judge_test.go`, `beginbatch_test.go`, `spec_test.go`, `bouncer_judge_test.go`, `fix_test.go`): settle whether the parent's envkit changes and the task's `ModelChoice` constructions leave a test redundant with another or a merged behavior uncovered, per `PATTERN-test-economy`."
  - "`cmd/lyx/testdata/test-budget.yaml`: settle whether the merged package budgets still match the tests both sides added, and whether the task's new top-level tests push a package past a row the parent also raised or lowered."
---

Seeded from the merge diff's overlap list.
Each entry names an interaction between a parent change and a task change, not a defect in either side alone.
