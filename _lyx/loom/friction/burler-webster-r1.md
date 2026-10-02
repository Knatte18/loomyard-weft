# Webster-Burler round 1: the worktree started with an uncommitted, test-breaking edit

The round started with an uncommitted edit to `internal/hubgeom/hubgeom.go` in the worktree, alongside an archived `round-1-review-20261002T091536Z.md`.
That points to an earlier attempt at this round that stopped partway through its fix.
The edit returned `PrimeName`'s error and broke 33 untagged wiring tests.
Nothing said whose edit it was, so the review had to be run against a `git archive` export of HEAD to judge the committed range cleanly.
The fix phase then replaced the edit rather than building on it.
A restarted round should start from a clean tree, or be told about leftover edits.
