# Plan round 1 — fixer report

## Changes

- **F1** (fixed as proposed): card 6's Intent gains a paragraph, before the `Short`/`Long` sentence, stating the commit as a new side effect, that it lets a post-validation edit plus a clean re-run become the committed copy past card 7's committed-copy check, and its bounds: a darn seed, `commitDiscussion`'s pathspec and a record passing the whole darn check, whose history check protects appended entries.
  The proposed text called the pathspec "the discussion directory alone"; `discussionCommitPathspec` (`internal/loomcli/wiring.go:331`) also adds the parent-review directory when it holds a file, so the paragraph says that.
- **F2** (fixed as proposed): card 7's `decisionFake` gains a field holding the bytes its commit leaves as the committed copy, which the commit uses when set.
  `TestDecisionVerb_Refusals` gains a post-commit row whose committed copy holds an `Added after Discussion` heading with no date, refused with `decisionWayEntryLost`, with `afterCommit` set.
  The `afterCommit` sentence now says "the rows that refuse after their commit", since more than one row sets it.
- **F3** (fixed as proposed): card 9's Intent says `DarnDescribeSpec`'s doc comment states the darn Describe reads the decision record beside the board entry and still reads no run record.
- **F4** (fixed as proposed): card 13 drops its Create group, the `cmd/lyx/testdata/test-budget.yaml` Edit bullet, the `writeBouncerJudge` and `haltingProducer` Uses bullets, and the three Intent sentences about the helper, the new test and the budget raise.
  `designWrongBouncerJudge` stays in Uses, as the extended `TestEscalation_DarnPerchRuleContinuesThenHardCapAcceptReachesDescribe` drives it.

`lyx loom validate-plan` over the edited plan returns ok, with only the informational `create-new-unit` findings.

## Disputed

None.

## Deferred

None.
