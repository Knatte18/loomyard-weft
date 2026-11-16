---
round: 3
focus:
  - "Cards 6 and 11 reset a local parent branch with `git reset --keep` under an archive-tag ownership: settle whether a planned symbol in `internal/fabricengine/destroy.go` executes it, checking containment, ownership, dirtiness and force in that order, with every caller routed through it."
  - "Cards 7 and 8 and the overview's `code-tip-is-warp-sha` decision rely on a `Warp-SHA` trailer on the records commit of a transition: settle whether the `Commit` closures of `battenCommitStatusDeps` (`internal/battencli/commitstatus.go`) and `commitStatusDepsGuardedBy` (`internal/loomcli/wiring.go`) write it, given that `fabricengine.CommitAnchoredPaths` passes the message through untouched."
  - "Card 8 removes the `Spawn` field of the commit-status deps: settle whether every non-test and test caller of that field, including `hubReadoptSeams` in `internal/battencli/readopt.go` and `TestPrePushCheckWiredIntoDeps` in `internal/loomcli/prepush_test.go`, is a target of card 8 or of an earlier card."
  - "Card 3's failed-push record is keyed by side and branch and written whole by `recordPushFailure` in `internal/fabricengine/failedpush.go`: settle whether two runs sharing one prime records branch each read as owed through card 7's `FailedPushesOwedBy`."
  - "Cards 6 and 7 move origin's code branch with a lease: settle whether each names the leased-move executor `updateRemoteBranch` in `internal/fabricengine/destroy.go`, its ownership and the dirtiness its request declares for a worktree holding uncommitted files."
---

Focus carries the open questions of round 2 only.
The settled remainder of the plan needs no re-review.
