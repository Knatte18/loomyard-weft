# Discussion stage has no sanctioned path to land a repository edit ahead of Plan

`discussion-lands-invariant` · Note · enhancement, loom

Imported from [issue #385](https://github.com/Knatte18/loomyard/issues/385).

During the Discussion stage of a loom task, the parent (`ly:orch`) relayed an operator request to edit `PATTERN.md` and `pattern/` directly in the task worktree, so the review rounds would read the revised rules.

The Discussion launch prompt allows only the two discussion files and no git.
An uncommitted repository edit would leave the tree dirty for the `PATTERN-verified-tree` check.
It took an extra round trip to get the operator's explicit instruction and a named committer (the driver) before the edit could be made.

Discussion has no sanctioned path for "land this invariant now, ahead of Plan".
If this should be supported, the Discussion launch prompt or the parent-review flow needs a carve-out that names who commits such an edit, instead of leaving it to an ad-hoc message exchange.

Provenance: task `test-writing-rules`, note `Discussion-Write.md`.
