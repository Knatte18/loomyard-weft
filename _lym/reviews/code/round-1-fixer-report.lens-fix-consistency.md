```yaml
lens: fix-consistency
outcome: reported
```

No findings that change a test result; the list of stale twins follows.

Task: check the F1 and F2 change set for twins left unchanged (lens fix-consistency).

- **LOW** — `internal/webstercli/verbs_test.go:1741`, `TestRecordBatchCmd_FabricSyncFailureWayForward`.
  - The test still commits the Create target at the root-less path `internal/only/new.go`, while the fixture plan's target now resolves to `backend/internal/only/new.go`.
  - It asserts the batch is terminal on disk, and it passes in the gate.
  - The sync refusal comes after `DoneChecks`, and terminal-on-disk holds whichever way the checks go.
  - The commit is therefore now stale against the fixture, and the test no longer runs the done path it did before. The `TestRecordBatchCmd_Envelope` commit was updated to `backend/internal/only/new.go`; this one was not.
  - Evidence: `grep` shows `backend/internal/only/new.go` only at lines 636 and 1091. The root-less path remains at 544, 565, 696, 859 and 1741.
- **LOW** — `internal/webstercli/verbs_test.go:696`, `TestRecordBatchCmd_FailedBatchEnvelope`. The same stale root-less commit. It asserts `batch_failed` and `card_amended`, and the card amendment forces the failure anyway, so the missing Create target adds an unasserted reason.
- **LOW** — `internal/webstercli/verbs_test.go:544` and `:565`, the `TestFingerprintRefusals_CarryPlanDrifted` seeds, and `:859`, `TestRecordBatchCmd_ReportArchivedEnvelope`. These commit the root-less path against the rooted fixture plan.
  - The fingerprint rows refuse before `DoneChecks`.
  - The archived-report row never reaches it.
  - Each commit is harmless and now disagrees with the fixture, so a reader sees a different target than the plan names.
- **LOW** — the plan rewrites at `verbs_test.go:352`, `:409`, `:577` and `:936` drop `root:` (`onlyCreatePlan` with no `Root`).
  - Rows that expect the overview to stay as recorded would trip over it; `TestRebaselineCmd_EditedCardOfFailedBatchThenRecover` already did and was fixed by setting `Root`.
  - The gate passes, because these rows expect a refusal (fingerprint, `plan_conflict`) or never read the overview. The `TestPersistentPreRunE_PlanDirAnchoredAtSubpath` doc comment still holds.

The `newVerbsFixture` doc comment still says the plan is a "single-batch plan fixture seeded under the fixture's own _lym/plan" and does not mention `root:`. That is accurate, just incomplete.

Calls made:
1. `grep` for `internal/only/new.go` and `backend/internal/only` in `internal/webstercli/*_test.go`. My context did not show the remaining committed-target sites.
2. `Read` of `verbs_test.go:1732-1761`. My context did not show the `TestRecordBatchCmd_FabricSyncFailureWayForward` body.
