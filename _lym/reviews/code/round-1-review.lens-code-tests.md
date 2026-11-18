```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**

- `fabricengine.CommittedFileDiffs`, `CommitMessages`: `TestCommittedFileDiffs_ListsFilesThatExistAtTheBase`, `TestCommitMessages_ListsWholeMessagesNewestFirst`. Pinned.
- `ParseJudgement` file and check rules: the new `TestParseJudgement` rows. These cover unknown, duplicate and missing ids, a bad ruling, an empty reason, none with checks, loosened without checks, an empty test, loosened text or cause, and a bad check ruling. Pinned. A judgement with no `files` key is pinned by `ValidCommandWithPassingCompareAndNoCases`.
- Drift-Verify under lines mode:
  - A listed file spawns the judge with no cell, and uncaused makes it Stuck with the ways forward. Pinned by `TestDriftVerify` "a widened limit with no trailer halts…".
  - A stored uncaused ruling halts without a judge, a trailer-only commit re-rules, and an unchanged range reuses the ruling. Pinned by the same test.
  - The review base is used, not the branch point. Pinned.
  - A no-match reason comes before an uncaused reason. Pinned.
  - A nil `ReviewBase` lists no file. Pinned.
- Drift-Verify under command mode, compare exit 0 with an unruled file: pinned. A failing compare together with files has no test, and neither has exit 0 with files already stored. See F-T1.
- `loosenedTrailerLines`: only the one-trailer path is pinned indirectly. See F-T2.
- Verdict `Loosened checks` lines: the new `TestVerdict` row. Pinned.
- A judged record from before this change, with no `files` key: none. Only the judgement parse is pinned. See F-T3.
- `Drift.ChangedFiles` and `Drift.CommitMessages` seam checks: the `VerifyNilChangedFiles` and `VerifyNilCommitMessages` rows in `TestDriftEntries`. Pinned.
- `Drift.ReviewBase` wired from the same value as `Env.ReviewBase`: `TestWire_ReviewKeysReachTheEnv`. Pinned. The `driftDeps` closures that call `fabric.CommittedFileDiffs` and `CommitMessages` are thin wrappers with no test.
- `pr_review_small_max`: the `config_test.go` default and override rows, and `TestPRReviewSmallMax_ReadsThePairsLayers` for the local layer. Pinned.
- Pull-request record keys: `TestRecord_RoundTripAndUpdate`, including the older record. Pinned.
- `ReadForEngine` plain includes: the `withfragment` rows, the codex row and the missing-fragment row. Pinned. A plain include that itself includes a plain fragment is left as it stands; no row.
- `LoomFragmentLoosenedCheck` registration: `TestStencils_TopLevelMarkers`. Pinned.
- The fix-step include: `fixStepIncludes` in `template_test.go` and `fix_test.go`. Pinned.
- Briefs that include the fragment:
  - The darn writer reads through `ReadForEngine` at `internal/darnengine/spec.go:164`, and the PR-Review and sampler briefs were switched to it. I could not confirm a test renders these three without a leftover `{{template`.
  - I found no test that renders the gatefix-darn body or the landing fixer through an include-resolving read. `internal/landingshed/fixer.go:106` only checks that the stencil exists.
  - The Webster and Weaver bodies read through `ReadSet`, and a render test for them is outside what I read.
  - See F-T4.
- PR-Review row, `settleSendBack` and `sendBack`:
  - The newest own entry being the marked file's reroute clears the mark, count, counted sets, ask and session, and starts a new session: `TestPRReview_Call` row "NewestEntryTheMarkedFilesReroute…" and `TestPRReview_Session` "AfterTheSendBackMarkIsCleared…". Pinned.
  - A re-step keeps the count: pinned.
  - No own entry, another file's reroute, and a cap-halt Stuck entry each keep the mark and reroute: pinned.
  - A dirty tree at Call halts with a way forward: pinned.
  - A wake reroutes on a clean tree, and a dirty tree halts and keeps the mark: `TestPRReview_Wait` rows. Pinned.
  - `checkWiring` for a nil `ShedStatus`: pinned.
  - A `ShedStatus` read error: none.
  - The real `ShedStatus` closure in `landingdeps.go`, which maps ErrNotExist to an empty status: none. See F-T5.
- `pr-findings` verb decisions, `TestPRFindingsVerb`:
  - Pinned: BLOCKING, count over the max, fix-here count lifting the total, and a small set staying.
  - Pinned: no `on_reroute`, a failing entry gate and a budget of 0 each answering fix-here.
  - Pinned: a second send-back numbered 2 with the first note kept.
  - Pinned: an empty friction dir, a dirty tree, an unparsable file and no record, each with a way forward.
  - Pinned: marked-record idempotence and the failed-commit rerun counting once.
  - Pinned: a read error of the dirty paths, of `pr_review_small_max` or of the edge.
  - Not covered: an entry gate that returns an error, and a target row missing from the recipe. See F-T6.
  - The "budget 0 means zero_rounds" premise is tested only as a literal `MaxBounces: 0`. See F-T6.
- `resolveRunID` exclusion for `pr-findings` (`arm.go:69`): none. See F-T7.
- The `prFindingsCmd` edge closure (`EngineRow`, `newShed`): none. The tapestry, darn and sampler rows are pinned by `TestRecipe_PRReviewReroutesToCodeBouncer`.
- `reviewInputs` seam: `TestReviewInputSeams` covers the fixer report as non-full, only the newest findings file as full, numeric order (10 over 2), and a non-matching name. Pinned. A missing scratch directory has no row.
- Bouncer, `TestBouncer_Amendment` rows:
  - Pinned: a full input over an unchanged subject, a round-0 settle record, `FullGeneration` set, both inputs recorded, and a legacy `review_input` record.
  - Pinned: the amendment note staying none for a full generation.
  - Not covered: a full input over a changed subject, `ReviewInputNote` being `none` for an amendment record, and an unreadable generation record. See F-T8.
- `Rubric` seam of the review producer: the two `TestBurlerReviewProducer_Call_ProfileAndOpts` subtests and `TestBurlerReviewEntry_RubricStencil` "RendersTheReviewInputNoteAtEachCall…". Pinned.
- `ReadRubric` `review_input` fill: `TestReadRubric` rows plus the allowlist in `TestLoomRubrics`. Pinned.
- Recipe edges: `TestRecipe_PRReviewReroutesToCodeBouncer` and `TestRouting_ProgressAt`. Pinned. Both are listed as contained in the redundancy report; I leave that economy call to the handler.
- Friction note per send-back: `TestPRFindingsVerb` `wantNote` and `wantNotes`. Pinned.

**Findings**

- F-T1, MEDIUM, design. Location: `internal/driftengine/verify.go` `storeCaseRulings` and `passCommand`.
  - Under command mode with a failing compare, the new `listed.newRulings` and `haltReasons(noMatch, uncaused)` path has no test.
  - The stored-rulings reuse path (`passWithoutJudge` after a passing compare) has no test either.
  - Only the passing compare with an unruled file is pinned.
  - Evidence: the command-mode subtest at `verify_test.go` (diff lines ~488–510) is the only one.
- F-T2, LOW, design. Location: `internal/driftengine/loosened.go` `loosenedTrailerLines`.
  - Nothing pins that only the last paragraph counts, that lines are sorted and de-duplicated, or that CRLF is handled.
  - The key stability that D3 relies on rests on these.
  - Only a message with a single trailer appears in the tests.
- F-T3, LOW, design. Location: `internal/driftengine/records.go` `JudgedRecord.Files`.
  - Acceptance criterion "a judgement record written before this change still reads" has a stored-record side, and no test reads a `judged.json` without `files` through `listFiles`.
  - Only the `judgement.yaml` parse side is pinned.
- F-T4, MEDIUM, design. Location: `contracts/stencils/darn/darn-template-write.md`, `gatefix/gatefix-body-darn.md`, `landing/landing-template-fixer.md`, `landing/landing-template-pr-review.md`, `sampler/sampler-template-agent.md`.
  - These briefs now carry a bare `{{template "loom-fragment-loosened-check"}}`.
  - Only the burler fix step has a test that renders it with the include supplied.
  - I confirmed `internal/darnengine/spec.go:164` reads through `ReadForEngine`.
  - I could not confirm that the landing fixer and gatefix-darn paths resolve the include, or that any test renders them.
  - If one reads through `Read`, the action reaches the filler unresolved, and no test would show it.
  - The fix is a render test per brief, as a row in an existing table, asserting that no `{{template` remains.
- F-T5, LOW, design. Location: `internal/loomcli/landingdeps.go` `ShedStatus` closure.
  - The ErrNotExist-to-empty-status mapping and the read-error path are untested.
  - `landingshed` tests inject a fake status, so a regression here would pass.
- F-T6, LOW, design. Location: `internal/loomcli/prfindings.go` `sendBackEdge`.
  - No row covers a target row missing from the recipe, or an entry gate that returns an error.
  - No test builds a real recipe whose Code-Review budget is 0 and shows `MaxBounces == 0` on the built Code-Bouncer.
  - That is the premise of the `zero_rounds` fix-here criterion.
- F-T7, MEDIUM, design. Location: `internal/loomcli/arm.go:69`.
  - The `pr-findings` exclusion in `resolveRunID` has no test.
  - Without it, the findings file path is taken as the run id and the verb breaks.
  - Add a row to the existing `resolveRunID` coverage with `verb == "pr-findings"` and a file argument, expecting `SelfRunID`.
- F-T8, LOW, design. Location: `internal/shedadapters/bouncer_amendment_test.go`.
  - There is no row for a full input over a changed subject, which goes through `startNextGeneration` from the changed path. The expected result is a full generation with no base.
  - There is no row for `ReviewInputNote` being `none` on an amendment record.
  - There is no row for an unreadable generation record.

**Tool calls**
1. `grep` for the readers of the include-carrying briefs and the `pr-findings` references. My context did not show how the darn, gatefix and landing-fixer stencils are read, nor any `pr-findings` test beyond `prfindings_test.go`.
2. `grep` for `stencilstore.Read*` calls in `darnengine`, `landingshed/fixer.go` and `gatefixer.go`, and for other `pr-findings` tests. It confirmed `ReadForEngine` in `darnengine/spec.go:164`, a bare `Read` existence check in `landingshed/fixer.go:106`, and no `pr-findings` test beyond `prfindings_test.go`.
