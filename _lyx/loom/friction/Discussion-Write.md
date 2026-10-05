# Discussion for test-writing-rules: parent asked for a repository edit the launch prompt forbids

After delivery, the parent (`ly:orch`) relayed an operator request to edit PATTERN.md and `pattern/` directly in the task worktree, so the review rounds would read the revised rules.
The Discussion launch prompt allows only the two discussion files and no git, and an uncommitted repository edit would leave the tree dirty for `PATTERN-verified-tree`.
It took an extra round trip to get the operator's explicit instruction and a named committer (the driver) before editing.

Discussion has no sanctioned path for "land this invariant now, ahead of Plan".
If the operator wants that again, the launch prompt or the parent-review flow needs a carve-out that names who commits such an edit, rather than leaving it to an ad-hoc message exchange.
