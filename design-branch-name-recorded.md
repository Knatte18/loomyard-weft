# fabric: branch_prefix is re-derived at teardown, so changing it strands every live pair

`branch-name-recorded` · Note · bug, fabric

Imported from [issue #59](https://github.com/Knatte18/loomyard/issues/59).

## Seen

The hub NORCE Models wants its LoomYard branches named `hanf/<slug>`, through fabric's `branch_prefix` (currently `""`, template).
The prefix cannot be set safely while any pair is live: the code re-derives the warp branch as `cfg.BranchPrefix + slug` from the current config at every later step instead of reading the branch the pair was created with:

- `internal/fabricengine/remove.go` (warpBranch := t.cfg.BranchPrefix + slug, the local branch check, the weft sibling via RecordsBranchName(t.cfg.BranchPrefix+slug)).
- `internal/fabricengine/pairparts.go` (remoteHeadTip on BranchPrefix+slug; slug recovered with CutPrefix(branch, cfg.BranchPrefix)).
- `internal/pairteardown/pullrequest.go` (taskBranch = cfg.BranchPrefix + slug when finding the PR).
- `internal/fabricengine/remoteteardown.go` (the job carries the current BranchPrefix).

So setting the prefix while beams and tasks run (here two beams and two task runs, one beam meant to live for weeks) makes their teardown, PR lookup and remove look for `hanf/<slug>` and miss the real `<slug>` branches.
A hub with a long-lived beam never has a moment with no live pair, so the prefix effectively cannot be introduced.

## Workaround

None applied: the prefix stays unset until this is fixed.

## Expected

A pair records its warp and weft branch names when it is created (in its origin record or pair record), and every later step (remove, teardown, PR lookup, pairs listing, cleanup) reads the recorded name.
`branch_prefix` then only names the branches of pairs created after the change, and can be set at any time.