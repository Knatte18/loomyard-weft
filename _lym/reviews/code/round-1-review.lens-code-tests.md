```yaml
lens: code-tests
outcome: reported
```

Scope: the code-tests lens. For each behaviour the diff adds or changes, I named the test row that would fail if it broke.

**Per-behaviour list**

1. **A failed post in `landingshed.PRReviewAsker.Ask` now ends `undelivered`, with "the request was not delivered: …" as its reason.**
   Pinned by `TestPRReviewAsker_Ask`, row "a refused post records undelivered with its reason" (`internal/landingshed/prreviewask_test.go:44`).
2. **A failed post in `driftengine.JudgeParentSeat.postJudgeAsk` now ends `undelivered`, with its reason.**
   Pinned by `TestParentAgreementGate`, row "a failed post records undelivered and passes" (`internal/driftengine/judgeparent_test.go:150`), which checks `wantState` and `wantReason`.
3. **A failed post in `shedadapters.parentAsker.post` now ends `undelivered`, with its reason.**
   Pinned by `TestBurlerReviewProducer_Call_ParentAskThatDoesNotGoOut`, row "a post error" (`internal/shedadapters/parentseat_test.go:549`).
4. **`DriftVerify.endedUnanswered` now counts an `undelivered` ask as ended unanswered: it halts under autonomy and otherwise stores the rulings.**
   Pinned by `TestDriftVerify`, rows `Undelivered_Autonomoustrue` and `Undelivered_Autonomousfalse` (`internal/driftengine/verify_test.go:1852`).
   If `ParentAskUndelivered` were dropped from the case list, the autonomous row would fail its halt assertion.
5. **`parentReviewAwaitVerb` now returns the reason of an `undelivered` ask.**
   Pinned by `TestParentReviewAwaitVerb`, row "an undelivered ask answers ready with its reason" (`internal/loomcli/parentreview_test.go:538`).
6. **`prReviewAsker` takes a strand claim: a non-orch strand in its own pane signs as that strand, and an empty claim signs as `LOOM`.**
   Pinned by `TestPRReviewAsker_AskReachesTheParentSignedAsTheCaller`, both rows (`internal/loomcli/prreviewask_integration_test.go:35-36`).
7. **`postSigned` signs strand-first and falls back to the program; it is shared by `notifyPrime` and `PostAsCaller`.**
   Pinned by `TestNotifyPrime_SignsAndPosts` (`internal/orchcli/notify_test.go:111-130`).
   Its rows cover:
   - a strand claim that creel accepts;
   - a claim refused outside the pane, which falls back to the program;
   - an orch claim, which signs as the program only;
   - an empty claim, which signs as the program only;
   - any other refusal, which returns an error.

   For `PostAsCaller` itself, behaviour 6's integration rows cover the claim and the no-claim case end to end.
8. **The refactor of `postNotice` into `notifyClientsOf` changes no behaviour.**
   `notifyPrime` keeps its result through `TestNotifyPrime_SignsAndPosts`.
   The existing `notifystamp_integration_test.go` tests exercise the client wiring.
9. **`parentReviewState.preRun` now passes the session's strand (`LYM_STRAND_NAME`) as the claim to `prReviewAsker`.**
   **No test pins this.** `TestParentReviewAskVerb` replaces `askPR` with `parentReviewPRAsker` or a stub (`internal/loomcli/parentreview_test.go:82, 438, 449`).
   The only test that reaches `preRun`, `cmd/lym.TestCLITree_EveryCommand`, checks the command tree, not the claim.
10. **`landingdeps` passes `""` as the claim, so the row's own ask still signs as `LOOM`.**
    Only indirectly pinned: the integration test's no-claim row checks `prReviewAsker` with an empty claim.
    No test checks that `pullRequestDepsGuardedBy`'s `askParentPR` passes `""`.
    This is a constant literal, so I raise no finding.

**Mutation survivors**

The survivors at `internal/driftengine/judgeparent.go:207` and `internal/shedadapters/parentseat.go:226` both negate `endErr != nil`.
The only effect is whether a `logger.Warn` fires, which no test can reasonably observe, so I raise no finding.
The state and reason that the focus directive asks about are pinned by behaviours 2 and 3.

**Findings**

- **severity:** MEDIUM
- **class:** design
- **location:** `internal/loomcli/parentreview.go:408` (`parentReviewState.preRun`, the `askPR` closure)
- **summary:** The behaviour issue #200 is about, that `parent-review ask` signs with the session's own strand, has no test.
  If `strand` were replaced by `""`, the bug would return (a program claim from the session pane) and every test would still pass.
- **evidence:**
  - `internal/loomcli/parentreview.go:408` holds `prReviewAsker(target, rec, strand)`.
  - The verb tests replace `askPR` (`internal/loomcli/parentreview_test.go:82, 438, 449`).
  - The reach report lists only `cmd/lym.TestCLITree_EveryCommand` for `parentReviewState.preRun`.
  - `TestPRReviewAsker_AskReachesTheParentSignedAsTheCaller` calls `prReviewAsker` directly with the claim (`internal/loomcli/prreviewask_integration_test.go:51`).
- **fix:** Move the `askPR` construction into a function that takes the strand, and have the integration test drive it, or add a row that asserts the claim `preRun` builds.

Tool calls:
1. One Bash call read `judgeparent.go:203-209`, `parentseat.go:222-228`, the rows of `TestNotifyPrime_SignsAndPosts`, and the `strandName`/`askPR` lines of `parentreview_test.go`.
   My context lacked the exact survivor lines, the coverage of `postSigned`'s fallback, and whether the ask-verb tests run the `askPR` wiring from `preRun`.
