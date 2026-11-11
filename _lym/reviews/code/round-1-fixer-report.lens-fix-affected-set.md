No findings.

Checked against my lens (each finding's own set, one search per finding):
- **F1:** `inReviewGroup` in `internal/loomcli/cli.go:290` skips exactly four groups: `circling`, `decisionGroup`, `parentReviewGroup` and `peersGroup`. The test slice now holds all four, so it covers every member.
- **F5, F7:** the other state-moving callers in `internal/burlerengine/parentreview.go` (`MarkParentAskPosted`, `ExpireParentAsk`, `AnswerParentAsk`) all route through `Ended()`. The new unknown-state rows reach `ExpireParentAsk` and `AnswerParentAsk`. `MarkParentAskPosted` is not covered, but it tests only for `ParentAskPendingPost`, so an unknown state cannot be moved there, and the review did not name it. The marker line has the same shape as the existing `//testtiming:keep` lines.
- **F8:** the other locked updates in `parentreview.go` (`MarkParentAskPosted`, `ExpireParentAsk`) already check `err` against `errParentAskUnchanged`, so only `AnswerParentAsk` had the misordering.

Left unchecked:
- **F2, F3, F4, F6:** each adds table rows to one verb test. No set premise, so no search.

Calls made:
1. `grep` on `internal/loomcli/cli.go` for `inReviewGroup` and the group names: the context did not show `cli.go`, so I needed the set the pre-run skips.
2. `grep` for non-test `state.UpdateJSON` callers: I needed other sites with the refusal-before-error ordering.
3. `grep` for `testtiming:keep` in test files: I needed the existing marker format.
