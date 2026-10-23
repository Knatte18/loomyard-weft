# Card 4 — pair-parts-reads

**Create:**
- `internal/fabricengine#PairParts`
- `internal/fabricengine#Topology.PairParts`
- `internal/fabricengine#Topology.TaskBranchOnOrigin`
- `internal/fabricengine_test#TestPairParts`

**Edit:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#Topology.inspectPair`

**Intent:**
Give fabric the reads the cancel needs (D2's nothing-to-cancel reading and D5's start refusal), in fabric's own vocabulary so callers outside the owner set never name a side.

`PairParts` reports, read locally with no network call, `TaskWorktree` (a registered linked worktree at the pair's location), `RecordsWorktree` (a directory at `RecordsWorktreePath`), `RecordsBranch` (the local sibling branch), `TaskBranch` (the local task branch) and `Launchers` (the pair's launcher directory), plus `Any() bool`, true when any of them is.
`Topology.PairParts` fills it from `inspectPair`, the same local reading `Remove` makes, so the two can never disagree on what is left.
`Topology.TaskBranchOnOrigin` reports whether the task branch exists on the warp origin (`remoteHeadTip`); a warp repo with no origin remote reads false, and an ls-remote failure is returned.

`TestPairParts` is a tier-1 integration test: after `Add` every part reads present; after `Remove` of a pair whose task branch is unlanded only `TaskBranch` reads present and `TaskBranchOnOrigin` reads the pushed branch; a slug never added reads nothing; a warp repo whose origin remote is removed reads `TaskBranchOnOrigin` false, and one whose origin URL points at a missing path returns an error.
`doc.go` names the pair-parts read and the origin task-branch read beside Remove's paragraph.

**ImpactSummary:** Read-only additions to `Topology`; no existing call changes.
