```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour: pinning test, or none**

| Behaviour | Pinned by |
|---|---|
| Raddle-File shapes, step order, beam release order | `loomshed/raddlefile_test.go` `EachShapeFilesOnceWithItsStepsInOrder` |
| Raddle-File sends back on a failed check, no approval, an edited proposal, a moved board or a board-write refusal, and deletes its outputs | `AFailureAtFilingSendsTheProposalBackAndDeletesItsOutputs` rows |
| Third return halts | `AThirdFailureHalts…` (a single test, not one row per return kind) |
| Retry after a failure in steps 4–7 skips the checks | `AFailureAfterTheFilingHaltsAndTheRetryFinishes…` |
| Rewrite of stage and part records for a changed proposal | `ACrashBeforeTheFilingRewrites…` |
| No filing until re-approved after a return | `AReturnedProposalIsNotFiledUntilApprovedAgain` |
| Raddle-Write mode recorded once, session count, fresh-entry reset, no commit on stuck | `loomshed/raddlewrite_test.go`, all rows |
| Release and end-entry writes | `TestReleaseTasks`, `TestEndRaddleEntry` |
| `FileRaddle` shapes, project level, dropped dependency, recheck refusal | `boardengine/raddlefile_test.go` rows and `CheckRefusalWritesNothing…` |
| Slug rule against the store (Sketch slug, filed slug, pruned project, rename, promotion, nil seam) | `TestSlugRule_RaddleStore` |
| Store replay: lands, lands beside a concurrent write, set aside on a refused or lost add, deeper path not claimed | `TestFabricReplay` Raddle rows |
| Promotion rule | `TestDiscussedPromotion`, `TestValidateLabels` |
| Start guard: no record, other Sketch, other `created`, rename target, own stamp | `TestRaddleStartRefusal`, `TestStartGuard_Refuse` |
| Proposal checks, including a part typed `sketch` and the project's `type` | `raddleparser/check_test.go` |
| Recheck | `recheck_test.go` |
| `raddle-propose` and `raddle-approve` | `loomcli/raddleverbs_test.go` |
| Beam hand-off and marker | `TestBeamUp`, `TestBeamUp_RerunIsIdempotent` |
| Record-Write seeds a stored record | `loomshed/recordwrite_test.go` (`StoredRecordIsWrittenVerbatim…`, read error) |
| Part splice | `partsplice_test.go` |
| Session-end hook, including stopped sessions | `sessionend_test.go` |
| Usage at every outcome | `TestRun_Finalize_ReportsUsageAndTimes` |
| Usage writer: per-model pricing, locate and read failure fallback, unknown cost | `shedadapters/sessionusage_test.go` |
| Codex split | `codexengine/usage_test.go` |
| Seed's stored-record flag | `loomcli/sharedbootstrap_test.go:230` |
| Landed-size record and its failure | `finalize_test.go:504-553` |
| Cost report cells, fit, too few runs, round usage record ignored | `reportengine/cost_test.go` |
| Raddle report | `reportengine/raddle_test.go` |
| `lym board raddle` | `boardcli/verbs_test.go` `stepRaddle` |
| Recipe rows, gate attempts, `on_stuck` routing | `loomrecipe/raddlerecipe_test.go` |

**Findings**

1. MEDIUM, design. A raddle run's finish never makes its entry a Sketch, which the recipe's `RaddleFile` row selects (D10).
   - Location: `internal/landingshed/finalize.go`, `internal/loomcli/landingdeps.go`.
   - Evidence: `grep` for `RaddleFile`, `MakeSketch` and `EndAsSketch` in `finalize_test.go`, `wiring_commitstatus_test.go`, `landingdeps` tests and `raddlerecipe_test.go` finds no row that asserts this. `raddlerecipe_test.go` checks only rows and routing.
   - The acceptance criterion "a finished raddle run never makes its entry a Sketch" is unpinned.

2. MEDIUM, design. Every run runner installs the usage recorder (D12, card 5).
   - Location: wiring that calls `shedadapters.UsageRecorder` and `Runner.SetSessionEnd`.
   - Evidence: `grep -rn "UsageRecorder\|SetSessionEnd" --include=*_test.go` hits only `sessionend_test.go`, `wait_test.go:1631` and `sessionusage_test.go:204`. No test builds a run's runner and checks the hook is set.
   - If the wiring dropped the hook on any run kind, no row would fail.

3. MEDIUM, design. `FileRaddle`'s done marker: a repeat call writes nothing when the store holds the Sketch's directory with the same `created` stamp and hash. This is what step 3's skip rests on, including after a part has landed and been pruned.
   - Location: `internal/boardengine/raddlefile.go:90-92`.
   - Evidence: the `t.Run` list in `boardengine/raddlefile_test.go` is `CheckRefusalWritesNothing…`, `OpenEntryEnds…` and `AbsentAndDone…`. The shape table (lines 107-139) has no done-marker or "already filed" row. The `loomshed` retry tests use a fake `StoreDone`, so they never reach this branch.
   - Related and also untested: a stale same-Sketch entry with another `created` stamp or another hash goes on to file.

4. LOW, design. A `Finalize` resumed past the merge has no kept parent head and so records no landed size, logged at Info (`finalize.go` `recordLandedSize`).
   - Evidence: `finalize_test.go` rows `LandedSizeFails` and the first-try record test cover the record and its failure. No row covers an empty `parentHeadMerged`.

5. LOW, design. `lym report cost` and `lym report raddle` at the CLI have only error rows (`reportcli/cli_test.go:38-40`).
   - Location: `internal/reportcli/cost.go` and `raddle.go`.
   - Evidence: the JSON and markdown output paths, which the raddle agent reads, are pinned only at engine level.
   - I did not check whether `costmarkdown.go` or `raddlemarkdown.go` are exercised by the engine tests.

6. LOW, design. `lym board promote --recipe raddle` through the CLI.
   - Evidence: `boardcli/verbs_test.go:79-83` exercises `--recipe darn` only. The raddle promotion is pinned at engine level by `TestDiscussedPromotion`.
   - The CLI-to-`PromoteTo` pass-through of the recipe is shared with the darn row, so this is a weak gap.

**Tool calls (3 of 6)**
1. `grep` of `^func Test` over the diff's test files. My context held no test files.
2. `grep` of row names in the key test files. Same reason.
3. `grep` for Seed's `run.json`, the CLI rows, promotion `--recipe`, the Sketch-finish row, `FileRaddle` subtests, usage wiring and raddle report rows. I needed to confirm the absence of each.
