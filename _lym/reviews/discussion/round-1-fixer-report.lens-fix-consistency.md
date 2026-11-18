```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set to the decision record and support log for consistency with sibling text (fix-consistency lens).

**BLOCKING.** D5's new level-with-origin rule conflicts with D7's new F18 rule, which re-breaks F2's landing case.
- Changed text in D5: "then replays the local records-only commits onto origin's tip as D7 replays them; it refuses only when a local non-records commit is unpushed."
- Changed text in D7: "rewrites a replayed commit's `Warp-SHA` trailer to its new parent only when every commit origin added is records-only; otherwise it stops as owed, naming origin's code commits."
- Evidence: D8 now says "it brings the local parent level with origin's parent tip by D5's rule" after GitHub merges a view. In that case origin always added a code commit, the squash or merge of the view. So with unpushed local parent records commits, D7's replay stops as owed. That contradicts D5's "refuses only when a local non-records commit is unpushed" and the new D8 acceptance row "with uncommitted and unpushed parent records, a pull-request landing and a local landing both complete".
- The two rules need reconciling. One option: the replay that brings a branch level skips the trailer rewrite, or rewrites it to the new parent whatever origin added, and stops only for the transition-tip case F18 targets.

**MEDIUM.** D8's push-time view refresh has no base now that the publish takes the pull request's target branch.
- Changed text: "The caller passes the branch the pull request targets as the base; the verb refuses when that branch differs from the branch's `parent_branch`…"
- The sibling bullet was left unchanged: "Fabric's push of a branch that has a local view ref refreshes the view". A fabric push knows no pull request target, so that bullet no longer says which base the refresh passes.
- It should say the refresh uses the origin record's `parent_branch`, or the base the view was last published against.

**LOW.** D7 cites a D2 docs requirement that D2 does not state.
- Changed text in D7's Bound: "A `git reset` or `git stash` in the prime can drop its uncommitted live records; the docs D2 requires warn of it."
- D2's docs sentence still reads only "describe cloning a one-repository hub and what it puts on code branches". Only the Review acceptance row was extended with the reset/stash hazard, so its twin in D2's body is stale.

**LOW.** D5 step 4 does not record the merge SHA that D7's new predicate depends on.
- Changed text in D7: "A merge commit fabric wrote is a merge commit carrying fabric's merge trailer whose SHA fabric recorded in the worktree's `.lym` when it wrote it."
- D5 step 4, unchanged, says only "then commits with fabric's merge trailer". It never says the merge records its SHA in `.lym`, so the writer side of the predicate is missing.

**LOW.** "Both its parent and itself" in D7 can be read as an intersection with D5's new definition.
- Changed text in D7: "whose changed paths all lie under the records paths (D5) of both its parent and itself".
- D5 already defines a commit's records paths as "the union of the anchors its own marker and its parent's marker name". Re-applying "both its parent and itself" either widens the set to the grandparent's anchor or reads as "under both sets", an intersection. Either way it disagrees with the D5 definition it cites.

Further tool calls: none.
