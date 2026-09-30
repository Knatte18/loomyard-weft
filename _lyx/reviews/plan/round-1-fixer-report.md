# Plan-Review round 1 — fixer report

All ten findings are fixed.
`lyx loom validate-plan` returns `ok: true`, with informational `create-new-unit` findings only.
The `approved:` key was not touched.

## Changes

- **F1 (card 15):** added `internal/shedrecipe/entries_batten.go`, `internal/shedrecipe/entries_batten_test.go` and `internal/battencli/wire_test.go` to the Edit list.
  Intent now says the `requireSeam` call and its doc comment name `InnerRun.ReadDecision`, and that both tests follow the rename.
  The ImpactSummary names shedrecipe's batten entry as the second production user.
- **F2 (card 12):** added `internal/landingshed/publish_verify_test.go` to the Edit list.
  Its passing-verify cases now expect `Done`, and `TestPublishVerify_ApprovalPathRunsNoVerify` is removed with the other approval cases.
- **F3 (card 10):** added `internal/shedbuild/fixture_test.go` to the Edit list.
  Its `testLandingDeps` fills `ApprovalPath`, `RejectionPath` and `TaskHead`, and its `newTestEnv` fills `ReworkSpec` and the `Rework` seams, so `TestBuild_EveryRegisteredEngineBuilds` builds both new engines.
- **F4 (card 11):** the loomrecipe `testLandingDeps` fills `ApprovalPath`, `RejectionPath` and `TaskHead`.
  Card 11 now states that `sequence_test.go` and `resume_test.go` need no change, and why.
- **F5 (card 16):** added `contracts/specs/loom-status-spec.md` as a Prosa target.
  Intent now covers the overview module-tree line, the approval-before-sync sentence and the batten restart parenthetical, and the status spec's awaiting-at-Publish comment.
- **F6 (card 6):** the runlevel test seeds a finished run, with batches complete, a done `outcome.yaml`, a `summary.md` and an integration report.
  It asserts the outcome and summary are archived and the report removed before Master spawns.
- **F7 (card 9):** step 2 now says round candidates come from listing `ReworkDir` in the working tree, and that each `record.json` is read at HEAD through `ReadCommitted`.
- **F8 (card 9):** in step 1, an absent record while a round is committed at HEAD now re-runs `Rebaseline` and returns `Done`.
  An absent record with no committed round stays `Stuck`, and both cases are in the test list.
- **F9 (card 12):** the ImpactSummary now names the Publish-Awaiting consumers and says batten is unaffected.
- **F10 (card 1):** the ImpactSummary now counts one production caller plus the test callers.

## Deferred

None.
