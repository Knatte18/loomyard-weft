# Hub store gc runs while other runs use the store

`store-gc-while-busy` · Note · enhancement, fabric

Since `git-robustness`, Remove runs a foreground `git gc --auto` on the shared hub store, which repacks it while other pairs' steps run against the same store.
`readGoGit` now absorbs a changed pack set for lyx's own reads, but the repack still costs time and I/O during other runs.
Seen in the PR-Gate review of PR #483.

Idea: skip the gc at Remove while other runs are in flight, and run it at the next teardown that finds none.
