# Fixing code review round 2: the reach turn named tested functions as unreached

The reach note listed `hubCancelDeps`, `Topology.SettleGonePairTeardown` and `Topology.RemoveLocal` as reached by no mapped test,
though each is called directly by a tier1 test, and the maps of both packages were the worktree's own and minutes old.
`lym gate reach` answered the same, so ruling it out took reading the tests and running them.
The caveat "may run it unseen" does not say why the map missed these tests.
