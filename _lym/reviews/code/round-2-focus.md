---
round: 2
focus:
  - "internal/fabricengine/pairbranch.go, PairTaskBranch: when only a bare-slug pair's origin branch is left and branch_prefix is set, settle whether the callers that run after the local parts are gone, namely closePairPullRequest in internal/pairteardown/pullrequest.go and hubTaskBranch in internal/battencli/readopt.go, name the branch the pair was made with, and whether any new method added for this keeps PairTaskBranch itself network-free."
  - "internal/fabricengine/remove.go, removePair and refuseWhenNothingRemains: settle whether one Remove asks origin at most once, and whether the nothing-local test counts every local part, launchers included, the same way in both places."
  - "internal/fabricengine/pairbranch.go, PairTaskBranch: settle whether a records worktree directory whose repository root cannot be resolved still lets inspectPair, RemoveRefusal, Remove, PairParts and DiscardTaskBranch complete for a half-torn pair, as removeSibling did before the diff."
  - "internal/fabricengine tests for a changed branch_prefix over a pair made under the empty prefix: settle whether each changed behavior is pinned, namely PairTaskBranch's rules (an unrelated branch ignored, detached HEAD, only the records worktree, only a local branch, prefixed and bare both local, nothing left), the origin lookups in Remove, RemoveRefusal and DiscardTaskBranch, and the branch-to-slug steps (TaskBranchSlugs, the remote warp sweep, cleanup orphan, the --parent check of Add)."
  - "internal/pairteardown and internal/battencli tests: settle whether a prefix set over a live bare-slug pair is pinned for the pull request lookup and for re-adoption's TaskBranch and TaskBranchGone seams."
  - "internal/fabricengine/template.yaml, destructivegaps_integration_test.go and remoteteardown.go: settle whether the branch_prefix description matches the decision record's rule that a pair keeps the branch name it was made with, and whether any comment names a signature or field the diff changed."
---

The focus lists where earlier fixes land new text, so a fresh reviewer reads it cold.
