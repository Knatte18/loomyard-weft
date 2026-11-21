```yaml
lens: code-tests
outcome: reported
```

Task: review the diff's tests under the code-tests lens.

**Each behavior the diff adds or changes, and the test that pins it**

1. `AppendDecision` writes one `Drops Create:` line per target after the Rationale paragraph, trims each target, and refuses an empty target or one with a backtick or newline. Pinned by `TestAppendDecision_DroppedCreates`.
2. `DroppedCreates` reads only added entries outside fences, a later title wins for the same target, and a missing or empty path reads as none. Pinned by `TestDroppedCreates`.
   The error for a heading that does not parse has no row. Low.
3. `dropsCreateRefusal`:
   - A batch of several entries is refused: pinned by `TestDecisionVerb_Refusals` "--drops-create on a call recording several entries".
   - A target with a backtick is refused: pinned by the row "--drops-create target holding a backtick".
   - An empty or whitespace target has no row. It shares the backtick row's condition, so it is a low gap.
4. The `--drops-create` value reaches `AddedDecision.DroppedCreates`. Pinned by `TestDecisionVerb_AppendsAndCommits` "parent label dropping two Create targets".
5. `PostBatchChecks` with `DecisionRecordPath` set skips a dropped Create's done-check, warns about it, and leaves the caller's cards untouched. Pinned by `TestPostBatchChecks_DroppedCreateSkipsItsDoneCheck`.
   The test drops every ref of the card. No row checks that a ref which is not dropped is still checked.
6. Webster's `postBatchInputs` and weaver's `checkRecordable` and `checkStoppedBatch` pass `DecisionRecordPath`: no test.
7. `createTargetWayForward` gives the same three ways forward from either verb. Pinned by `TestFailCardNotDone_WayForward` (CreateNotDoneOffersAmendDropAndRecover, RenameNotDone…, CreateNotDoneAfterARecoveryOffersTheSameWays, ExhaustedBatchWithAnUnrenderedAmendment…) and by `TestRecordBatch_DoneChecksBlockOnUnresolvedCreate` and `TestRecoverBatch_TerminalRunsTheSamePostBatchChecksAsRecordBatch`.
8. `verifyGateStuckFailure`:
   - A cut entry's findings name the report: pinned by the "long entry findings text…" and "non-drift entry…" rows.
   - An uncut entry names "the full report is": pinned by "a run-level entry's failure names…".
   - The drift entry adds the `drift amend` route and other entries do not: pinned by the drift rows and the non-drift row.
9. `renderParentDecisions` numbers the decisions in record order. Pinned by `TestRenderPrompts` "…decisions…" (inOrder). The `none` rendering for no decision lost its pin (finding 3).
10. `CheckReviewRound`:
    - Amendment exemption by stem, by slug, the short-stem guard, an unnamed card refused, still BLOCKING-only, a carried finding moved onto an amended card: all pinned by `TestCheckReviewRound` rows.
    - A `webster` finding passes with a halt reason, is refused without one, and is refused below BLOCKING: pinned by the three NewWebsterFinding… rows.
11. Prompt rendering: the amending mark and its paragraph, and the halt-reason block or its absence. Pinned by `TestComposePrompt_PriorRounds` (AmendingEntryIsMarkedAndExplained, NoAmendingEntryHasNoParagraph, HaltReasonIsQuotedAsABlockingFinding, NoHaltReasonHasNoBlock).
12. `BuildRoundContext`:
    - The amending texts, including entries already in the earlier snapshot, none without a record, the halt reason from its file, and sources left empty reading nothing: all pinned by `TestBuildRoundContext` rows.
    - The error paths have no row: the amendment reader's error, the entry-count mismatch, and a halt read error. Low.
13. The `runplan` halt-reason write, read, remove, and remove when absent. Pinned by `TestHaltReason_RoundTrip`.
14. `WebsterProducer` writes the halt reason on each Stuck and clears it on Done. Pinned by `TestWebsterProducer_OutcomeStuck`, `TestWebsterProducer_PendingAuditFindingsIsStuck` and `TestWebsterProducer_OutcomeDone`.
    The survivors at `webster.go:64` and `webster.go:71` sit in log-only branches, so they are no finding.
15. `BurlerReviewProducer.Call` reads the halt file and sets `profile.HaltReason` and `probeProfile.HaltReason`: no test. The mutants at `burlerreview.go:360`, `379` and `383` survived.
16. `shedrecipe` wires the amendment reader and the halt directory for Plan-Burler-Review alone, through `amendmentTitlesAt`: no test.
17. `burlercli` validate-review fills the amendment and conflict sources for the plan run directory: no test.

**Findings**

1. MEDIUM, design: no test runs record-batch, recover-batch or weaver with a decision that drops a Create.
   - Location: `internal/websterengine/recordbatch.go:902` (`postBatchInputs` `DecisionRecordPath`), with `internal/weaverengine/record.go:170` and `internal/weaverengine/resume.go:448`.
   - Deleting the field at any of these sites leaves every test green. The issue's core behavior, a batch that no longer blocks on a dropped Create, is pinned only inside `runplan`.
   - Evidence: the diff pieces 012 and 013 add only the field. A grep of the websterengine and weaverengine tests finds "drops-create" only in the way-forward strings; `recordbatch_test.go:1619` and `recoverbatch_test.go:720` hold no decision record that drops anything.
2. MEDIUM, design: the review producer's halt-reason plumbing has no test.
   - Location: `internal/shedadapters/burlerreview.go:359-383`.
   - The surviving mutants at `burlerreview.go:360`, `379` and `383` show that no test puts a halt file in `HaltConflictDir` and checks that the reviewer prompt quotes it. The changed-file exemption the round check gives also depends on this path.
   - Evidence: a grep of `burlerreview*_test.go` for `HaltConflictDir`, `HaltReason` and `AmendmentTitles` finds nothing.
3. MEDIUM, design: a removed assertion carries no `Loosened-Check:` trailer, and the `none` output is now unpinned.
   - Location: `internal/websterengine/template_test.go:908-911`, the "master renders none for no parent decision" row.
   - The diff removed the assertion that a prompt with no decision equals one rendered with `none`, and no commit in the range carries a `Loosened-Check:` trailer.
   - The new row checks only that `Decision 1:` and the marker are absent. A `renderParentDecisions` that returned "" for no decision would pass.
   - Fix: assert the rendered decisions section reads `none`.
   - Evidence: the 013 diff at `template_test.go` lines 908-911 and the `git log` messages.
4. LOW, design: no test checks that only Plan-Burler-Review gets the amendment reader and the halt directory.
   - Location: `internal/shedrecipe/entries_burler.go:185-188` and `internal/burlercli/validate.go:122-128`.
   - The recipe guard on `loomshed.NamePlanBurlerReview` and validate-review's plan run-directory branch have no test. Without one, Code-Burler-Review could start honoring the plan's halt reason, or the plan review could stop, unnoticed.
   - Evidence: a grep of `shedrecipe/*_test.go` and `burlercli/*_test.go` for `AmendmentTitles`, `HaltConflictDir`, `planReviewRunSubdir` and `webster-halt` finds nothing.
5. LOW, design: `withoutDroppedCreates` is tested only with every ref of the card dropped.
   - Location: `internal/runplan/postbatch_test.go:241-249`.
   - An implementation that dropped the whole Create group whenever one ref is dropped would pass.
   - Fix: add a ref that is not dropped and assert its create-not-done still blocks.
   - Evidence: `postbatch_test.go:243-249`.
6. LOW, design: two error paths at file-read boundaries have no row.
   - Location: `internal/shedadapters/roundcontext.go:146-177` (`amendingEntryTexts`: the reader's error and the "changed while read" mismatch) and `internal/discussionparser/append.go:510-512` (`DroppedCreates`: the error for a heading that does not parse).
   - Evidence: `TestBuildRoundContext` and `TestDroppedCreates` in the 009 and 006 diffs.

Tool calls:
1. A grep of the `burlercli`, websterengine, weaverengine, shedadapters, shedrecipe and loomcli test files for the plan run directory, for "Drops Create" or "drops-create" usage, and for the halt and amendment fields. My context held the diff's tests but not whether other existing tests exercise these paths.
