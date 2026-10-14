# Lens review

All four forks returned a report.

- row-reachability + forced-edits, BLOCKING, card 11: `sessionHarness.spawnInput` builds a `PromptInput` without `MeasureStencil` and reaches `Prompt` through `Session.Begin` (`internal/alignment/session.go:204`). Fixed: the helper is listed on card 11 and sets `MeasureLoom`.
- criteria-coverage, MEDIUM, card 13 (D9 agent reject routes to Darn-Write): Rejected. PR-Align writes `RejectionPath: loomengine.LoomRejectionPath(l)` for both recipes (`internal/loomcli/landingdeps.go`, `landingDeps`), the file `pendingRejectionReader` reads (`internal/loomcli/wiring.go:304`) behind the darn writer's `ReadRejection` seam (`internal/loomcli/darnwiring.go:100`). The only darn-specific link, PR-Gate's bounce to Darn-Write, is pinned by `TestNewDarn_Routing`. The reject record and second-reject rule are pinned by `TestPRAlign`, and the rendering by `TestDarnWrite_Call`. Card 12 now states the path link.
- criteria-coverage, MEDIUM, card 12 (D9 decision add during the darn agent turns approve into an escalation): Fixed: the Config test's new subtest asserts the darn deps' `Snapshot` changes after an appended entry.
- criteria-coverage, MEDIUM, card 10 (the fixer defers an unrecorded correction): Fixed. The rubric's fix text frames the change as outside the fixer's write surface, and a new claim on `burler-step-fix.md` pins "is deferred with that reason, never disputed".
- criteria-coverage + row-reachability, MEDIUM, card 7 (read-error and post-commit read branches unreached; no `readRecordErr` on the fake): Fixed. The fake gains `readRecordErr`, history and post-commit errors and a darn-call count, and rows reach each error branch.
- test-economy, LOW, card 12 (new top-level test should be a subtest; render duplicates TestPrompt): Fixed. The test was folded into `TestLandingAlignmentDeps_Config` and the render assertion dropped.
- row-reachability, LOW, card 12 (render needs Point and paths filled): Fixed by dropping the render, per the finding above.
- test-economy, LOW, card 7 (darn task-session row repeats the caller branch): Fixed. The existing row asserts that the darn seams are never called.
- test-economy, LOW, card 7 (darn finding-after-append row never exercises the restore): Fixed: the row was dropped.
- test-economy, LOW, card 4 (missing-section subtest repeats an existing one): Fixed. The subtest asserts only that `history` is never called.
- criteria-coverage, LOW, card 8 (respawn rule and no-interview line unpinned): Fixed: claims were added.
- criteria-coverage, LOW, card 4 (history-not-latest property untested): Fixed: the dropped-entry subtest also runs with the newest history version being the dropping copy.
- criteria-coverage, LOW, card 10 (fasit assertion too narrow): Fixed: the assertion widened to the claim, class, severity and leave-the-code clauses.
- criteria-coverage, LOW, card 11 (escalate Go path unnamed): Fixed: the card names `TestPRAlign`'s escalate row.
- forced-edits, LOW, card 6 (testtiming reason, parity_test and validate.go headers stale): Fixed: all three are named on card 6.
- forced-edits, LOW, card 1 ("both" test doc, prompt.go marker count): Fixed. Both docs are updated; the test name stays.
- forced-edits, LOW, card 1 (Discussion-Review rubric does not point at the spec): Fixed: the rubric line also names the spec.
