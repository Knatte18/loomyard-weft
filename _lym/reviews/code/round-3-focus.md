---
round: 3
focus:
  - "Settle whether a motif run (recipe running Discussion alone, ending with Discussion-Bouncer done and no Finalize row, so no landing commit) is handled soundly by batten's Run-Shed, Land and Worktree-Teardown steps: read internal/battenshed/land.go (the pull of the prime, the deploy command run when the parent is a deploy branch, the records-stop notice worded 'landed') and the teardown request that sets Landed: true, and decide whether any of them acts as if code landed, and whether a test in internal/battenshed or internal/battencli drives that end state."
  - "Settle whether the commit-status seam is pinned at both of its construction sites in internal/loomcli/wiring.go, wireLightweight and wire: for each site, would replacing the call with loomCommitStatusDeps(location, c.runID) leave FinishedStatus empty for a motif run and fail some test?"
  - "Settle which failure returns of seedDiscussedRecord in internal/loomcli/discussionrecord.go (board open, DecisionRecord read, discussion directory creation, record write, support-log stat, support-log write, commit) have a row in TestSeedDiscussedRecord_Real in internal/loomcli/discussionrecord_integration_test.go, and whether each uncovered one returns a distinct error the gate turns into a failed run."
---

The focus entries follow the three findings still open after round 2.
