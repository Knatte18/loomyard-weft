# Lens review

Every fork returned a report.

- forced-edits, BLOCKING, the new `GateConfig` seams trip `envkit.NilSeams`: fixed. Card 6 adds `ParentReview.Aligner` and `ParentReview.RenderAlignmentBrief` to `internal/testkit/envkit#nilLegal`, which already lists the other five `ParentReview.*` seams (`internal/testkit/envkit/envkit.go:59-63`).
- forced-edits, MEDIUM, `Landing.Alignment.*` nil in the loom and batten seam walks: fixed. Card 9 lists the three seams in both `intentionallyNil` maps, beside the sibling `Landing.*` entries (`internal/loomcli/wiring_seams_test.go:14-26`, `internal/battencli/wire_seams_test.go:47-50`).
- forced-edits, LOW, `statusCommitPathspec`'s doc comment enumerates the staged dirs: fixed. Card 7 says it names the alignment dir.
- forced-edits, LOW, `CommitDetail` diffs each commit: rejected. The range is the parent's commits since the run's base, a handful per run, and the read stays in-process through `gitrepo`.
- criteria-coverage and row-reachability, MEDIUM, `hubPeersDeps`' real reads untested: fixed. Card 3 adds `TestHubPeersDeps_OverRealHub` over `hubforge`, pinning the changed paths, the parent-commit range direction, and a missing pair and a missing branch.
- criteria-coverage, MEDIUM, the PR-side off switch is untested: fixed. Card 9 adds `TestLandingAlignmentDeps_Config`.
- criteria-coverage, LOW, `final` with no reviewer passes an agent reject: fixed. Card 6 checks `noReviewer` in `final` only for a round with no verdict when `Aligner` is set, and adds a row for it.
- criteria-coverage, LOW, goto cannot cancel a running row: fixed. Card 9's Intent now says a goto reaches `PR-Align` through pause, and states the closed-attempt consequence.
- criteria-coverage, LOW, no missing-branch row in `TestPeersVerb`: fixed, the row now carries a missing branch too.
- row-reachability, MEDIUM, a two-key coherence row does not fit `TestLoadConfig_Refuses`' table: fixed. Card 1 adds an `extra` field written through `writeLoomConfigWithKeys`.
- row-reachability, MEDIUM, `TestParentReviewAligner` cannot inject a fake shuttle: fixed. `newParentReviewAligner` takes an `alignment.Shuttle`, and `wire` passes `alignment.RunnerShuttle(runner)`.
- row-reachability, MEDIUM, the snapshot rows need a real hub board: fixed. `alignmentSnapshot` takes a `notes` closure, and production passes `hubBoardTasks`.
- row-reachability, MEDIUM, `TestSession`'s died and missing rows race the watch goroutine: fixed. Card 5 adds `Session.watchDone`, and those rows wait on it.
- row-reachability, LOW, the `Notified` re-call assertion cannot fail: fixed, the row is replaced by notify and no-notify rows.
- test-economy, LOW, per-key local refusal rows repeat the generic guard check: fixed. Card 1 keeps one `skip_alignment` row per test plus the accepted timeout row.
- test-economy, LOW, an accept case sits in the refusal test: fixed. It moves to `TestLoadConfig_Loads`.
- test-economy, LOW, fold `TestApproveVerb_RemoveRejectionFailureWritesNoApproval` into `TestApproveVerb_Refusals`: rejected. Every row of that table must name a way forward (`internal/loomcli/approve_test.go`, the "names no way forward" check), and a write failure has none.
- test-economy, LOW, two `TestPRAlign` rows reach the closed-attempt branch: fixed, merged with the notify fix above.
- test-economy, LOW, `TestParentReviewAligner`'s timeout row repeats card 5: fixed, the row is dropped.
