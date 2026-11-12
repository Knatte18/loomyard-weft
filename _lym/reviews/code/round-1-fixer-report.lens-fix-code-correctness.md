Lens fix-code-correctness: I ran the added or changed rows and branches step by step and checked that each added test fails with its defect in place. I found one problem, and it is in a test the fix weakened.

**Findings**

- **LOW** — `internal/burlerengine/parentseat_test.go`, `TestParentRulesBlock`, after the fix dropped the `origin: parent` / `lym loom parent-review await` loop.
  - **Summary:** The test now passes when the parent block never reaches the composed step.
  - **Evidence:**
    - The remaining check is `strings.Replace(step, block, "", 1) == plainStep`. If the `parent_rules` marker is left unfilled, `step == plainStep`, so `Replace` removes nothing and the check still passes.
    - Nothing else in the test asserts that `block` is present in `step`.
    - My grep of `internal/burlerengine/*_test.go` for `origin: parent`, `parent-review await`, `parent_rules` and `parentRulesBlock` found no assertion on that prose in `TestEngine_Review_TableShape` or `TestComposePrompt_ClusterRules`.
    - The review claimed those two tests cover it. Their coverage is therefore unconfirmed.
  - **Suggested fix:** Add `strings.Contains(step, block)` next to the `HasPrefix` check. That asserts the block is composed without pinning its static prose.

**Walkthroughs that held**

- Nanosecond-seed row: nanosecond precision asks and passes; second truncation does not ask and fails the row.
- Unreadable-history row: the error is logged and the timeout ask yields no ask; treating the error as rewritten would fail the row.
- Missing-report row: `ReadParentReport` errors and the round asks again; treating the error as no findings would fail the row.
- `TestRoundSeed` fails under second truncation.
- `TestBurlerReviewProducer_Call_ParentRecordOrSeedUnreadable`: both rows end with `ParentAsk` 0, no `Resolve` calls and no posts.
- Coverage-gate rows: a missing review returns a failing result containing "read review file"; a missing report returns an error and the ask stays done.
- `review_test.go` assertion only strengthens the gate-entry check; removing `TestParentCoverageGateEntry_Shape` leaves no dangling references; the `loomcli` header edit is comment-only.

**Tool calls**
1. `grep` over `internal/burlerengine/*_test.go`, to check whether the dropped prose assertions are covered elsewhere. My context lacked the bodies of `TestEngine_Review_TableShape` and `TestComposePrompt_ClusterRules`.
