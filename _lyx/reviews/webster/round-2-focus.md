---
round: 2
focus:
  - "Comment convention in test files: CONSTRAINTS.md's Comment Line-Break Convention (one sentence per line, further breaks only at independent-clause boundaries) governs every new or changed Go comment, test files included. Check whether the diff's _test.go comments comply: the file headers of internal/shedadapters/bouncer_verdict_test.go and bouncer_circling_test.go, writeFactsReview in internal/shedadapters/bouncerfacts_test.go, TestBouncerEntry_EnvJudgeFallback in internal/shedrecipe/entries_bouncer_test.go, TestResolveJudge and TestResolveJudge_MalformedSpec in internal/loomengine/review_test.go, TestLoadConfig_MalformedJudgeSpec in internal/loomengine/config_test.go, and the header of internal/burlerengine/verdict_test.go. Also check the sentences that straddle an APPROVED-to-CONVERGED or BLOCKING-to-CONTINUE changed line in internal/shedadapters/bouncer_clear_test.go, bouncer_commit_test.go, bouncer_replay_test.go, bouncer_judge_test.go, burler_test.go (writeJudgedRound), and internal/loomrecipe/fixture_test.go, resume_test.go, sequence_test.go. Untouched legacy lines that no changed sentence crosses stay out of scope."
  - "Confirm the round-1 fix commits 273745f04 (envkit nilLegal SegmentBounces), f9b0a9288 (gofmt), 8e7defc94 (non-test comment reflow), cb3cd41ee (circlingStatusReader in internal/loomcli/circling.go, which creates the lock directory when the status file exists) and 4b3f38814 (CONSTRAINTS.md tight list) introduced no regression. Check that go test ./... and gofmt -l over the changed Go files stay clean."
---

The only defect still open is the comment convention in test files, which the round-1 sweep scoped out.
The second entry guards against regressions from the fix commits and is not a suspected defect.
