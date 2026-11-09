```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, covering per behaviour the test that pins it, or none.

**Per behaviour, the pinning test**

- **Marker filled only for a rubric carrying it.** Pinned by `TestReadRubric` in `internal/shedadapters/rubric_test.go`: the "renders the redundancy report note" row, and the "a rubric without the redundancy marker never reads the report" row, whose reader panics if called.
- **Nil `Env.RedundancyReport` or `RubricNotes.RedundancyReport` renders `none`.** Pinned by the "no redundancy report renders none" row of `TestReadRubric`.
- **A blank answer from the reader renders `none`.** No row. `noteOrNone(read())` in `internal/shedadapters/rubric.go:90` is only reached with a non-blank string or a nil reader.
- **Repeated marker.** No row. `slices.Contains` and `stencil.Fill` handle it, and it is not a separate behaviour.
- **Code-Bouncer call site.** Seed and judge prompts are pinned by `TestBouncer_ClusterExcludesReachesSeedAndJudgePrompts`. The `Env` to `BouncerConfig` hand-off is pinned by `TestBouncerEntry_HappyPath`.
- **Code-Burler-Review call site.** Pinned by `TestBurlerReviewEntry_RubricStencil`, subtest `FillsTheRedundancyReportNote`.
- **The `Env.RedundancyReport` closure is wired by `loomcli`.** Pinned by `TestWire_ReviewKeysReachTheEnv`, but only for the no-report line.
- **Rendering of the note.** `TestRenderRedundancyReportNote` has rows for no report, red gate naming the gate key, judged nothing, match failed, other tree, and match listing every test and error. The red-gate row also covers the "red-tier report never matches" part, since `matchesHead` is `unreached`.
- **The rubric's allowlist and marker carry.** Pinned by `TestLoomRubrics` in `contracts/stencils/rubric_test.go`: the allowlist, and the marker carried only by the code-review rubric.
- **The match holds after a commit touching only run records, and fails on an edit to a judged Go file or to a taken test.** Pinned by `TestRedundancyStep`, subtest `TheReportMatchesACommitOfItsTreeUntilATakenTestChanges`, in `internal/gatecli/redundancy_integration_test.go:318-345`.
- **The match holds when the diff also changes a `tier2` test.** None. No row in that subtest edits a tier2 test or asserts that the match survives one. `TestDropsTier2` appears only as a not-judged line in the report.
- **The closure matches against the worktree's HEAD.** None. The glue in `redundancyReportNote` (`internal/loomcli/redundancynote.go:22-40`) is untested. That glue is `ReadRedundancyReport`, `fabricengine.Open`, `HeadSHA`, then `RedundancyReportMatches`. Passing `HeadSHA` as `rev` is only checked indirectly. `matchesHead` is a fake in `TestRenderRedundancyReportNote`, and the wiring test only reaches the not-found branch.
- **The read-error line, "the report could not be read".** None. `internal/loomcli/redundancynote.go:26` is unreached.

**Findings**

1. MEDIUM, `design`, `internal/gatecli/redundancy_integration_test.go:318-345`. The decision record's acceptance criterion for D8 says the marker holds the findings "including when the diff also changes a `tier2` test", and no test row pins it. A change that made a changed tier2 test break the match would pass every test. Evidence: the subtest lists only run-record, Go-file and taken-test commits, and `TestDropsTier2` is not edited anywhere in lines 318-345. The fix is a row that commits an edit to a tier2 test file and expects the match to stay true.
2. LOW, `design`, `internal/loomcli/redundancynote.go:22-40`. The closure's read-error branch and the HEAD-match path are unpinned. The closure is only exercised for the no-report case in `internal/loomcli/wiring_test.go:943-947`. Without such a test, the acceptance criterion (a matching report yields the findings, and a red-tier report always yields the no-report line naming the gate key) is not tested end to end through `loomcli`. A small test with a real report file and a worktree would cover it, or an error row for a malformed report file.
3. LOW, `design`, `internal/shedadapters/rubric.go:90`. The "reader returns a blank string renders `none`" behaviour has no `TestReadRubric` row. The fix is one row with `func() string { return " " }` expecting `none`.

**Tool calls**

1. `grep` over `internal/gatecli/redundancy_integration_test.go` for tier2 and `wantMatch` usage. My context lacked the test's content beyond its diff.
2. `Read` of `internal/gatecli/redundancy_integration_test.go` lines 296-352, to see which commits the match subtest covers. My context held only the diff hunk.
