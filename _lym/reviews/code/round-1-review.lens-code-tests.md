```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing each behaviour the diff adds or changes against the test that pins it.

**Behaviour → pinning test**
- Fix check refusals (D1: missing, repeated or invented id; dispute ground, rebuttal and `<path>:<line>` evidence; reason; `spawned`; ruling scope; frontmatter parse): `TestCheckFixerReport` (`internal/burlerengine/fixerreport_test.go:44-71`).
- Skipped reports carry `spawned: false` and parse: `fixerreport_test.go:95`.
- Fixer-report gate entry (3 attempts, `PassOnCap`, wired last in the fix table, passes on the cap, a re-prompt can repair): `internal/burlerengine/fix_test.go` gate and outcome rows.
- `decisions_routed` recorded by the fix producer: `internal/shedadapters/burlerfix_test.go:266` (notifier and not-routed rows).
  - Gap: the `escalated: true` input is declared and assigned (`spec.decisionsEscalated = tt.escalated`), but no visible row sets it.
- Round checks (D3, D4): `TestCheckReviewRound` (`internal/burlerengine/roundcheck_test.go:96-155`) covers prior one-to-one, withdrawn, open answer, held statuses and quoting, carries rules, the new-finding bounds and the parent exemption.
- `ParseCurrentReview` admits `prior` and `carries`: `internal/burlerengine/verdict_test.go` ("prior and carries pass the shape checks").
- Prior-findings block rendering for Claude and Codex: `prompt_test.go` "round context" rows.
- `BuildRoundContext` (D2): `TestBuildRoundContext` (`internal/shedadapters/roundcontext_test.go`) covers completeness, held marks, the digest diff including a fold, the amendment generation, the changes file and the parent report.
- Snapshot record (D5): `TestBurlerReviewProducer_Call_RecordsRoundSnapshot`.
- Changes file and amendment behaviour: `burlerreview_test.go` rows.
- `TargetSnapshot` (overlay and source): `TestTargetSnapshot`.
- Fabric verbs: `TestFabric_WorktreeTreeID` (`internal/fabricengine/taskpatch_integration_test.go`).
- Stalemate check (D6, count, reset, yield, replay, auto-accept): `TestBouncer_Stalemate`.
- Decision round records the post-fix id: `bouncer_decision_test.go` subtest.
- Notices name the verbs: `bouncer_circling_test.go` rows and `bouncer_decision_test.go:231-238`.
- Arbiter eligibility: `arbiterstop_test.go` "pending stalemate".
- `accept` over a stalemate record: `circling_test.go:114`.
- Config default, 0 off, negative refused: `internal/loomengine/config_test.go` rows.
- Bouncer config wiring (recipe → `StalemateRounds`): `internal/shedrecipe/entries_bouncer_test.go` ("UnchangedRoundsEscalateAsStalemateOnlyWhenEnabled").
- `validate-review` (run directory, outside it, no repository, outside the run root, builder error): `TestValidateReviewVerb_RoundChecks`.
- Review-entry snapshot wiring: `entries_burlerhalves_test.go` ("OverlayTargetRecordsTheRoundSnapshot").

**Findings**

1. LOW, class `design`. `lym loom circling continue` over a `stalemate` escalation has no test.
   - D6 and the Acceptance criteria require that `continue` settles a stalemate and keeps its hard-cap refusal.
   - Evidence: `internal/shedadapters/circling_test.go:114` covers `accept` only. The grep found no `stalemate` in `internal/loomcli/*_test.go`. The Bouncer test "AfterAContinueTheNextUnchangedRoundAloneDoesNotEscalate" covers the count restart, not the verb's recording or its hard-cap refusal.
   - Fix: add a `continue` row over a stalemate record, below and at the hard cap, in the circling verb tests.

2. LOW, class `design`. The `escalated: true` branch of `DecisionsRouted` has no row.
   - Evidence: `internal/shedadapters/burlerfix_test.go` declares `escalated` and `wantRouted`, but the visible rows are one with a notifier and one with neither.
   - Fix: add a row with `escalated: true` and `wantRouted: true`.

3. LOW, class `design`. The source-scope closure wiring has no test.
   - The `ReviewTreeID` and `ReviewTreeChanges` closures in `internal/loomcli/wiring.go` and `internal/battencli/wire.go` have no test that exercises them. The grep found `ReviewTreeID` in tests only at `entries_bouncer_test.go:1319`.
   - Fix: none required if the closures count as thin wiring. Otherwise add a wiring test.

**Not judged:** the stencil and review-format prose is exempt. I did not check test economy against the redundancy report or the `testtiming:keep` marks. I also did not check whether a test pins the `validate-review` help text.

**Tool calls (3 of 6)**
1. Listed test function and row names across the diff's test files. My context held the test diffs only by name from the manifest, not their contents.
2. Listed row names in the `roundcheck`, `fixerreport` and `validate` tests, and the diffs of `burlerfix`, `fix`, `arbiterstop` and `circling` tests. This was needed to confirm row coverage.
3. Grepped for `decision add` notice assertions, `stalemate` in the `loomcli` and circling tests, and the tree-closure usages. This was needed to settle the gaps above.
