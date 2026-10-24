# Card 4 — add-from-parent

**Edit:**
- `internal/fabricengine#AddOptions`
- `internal/fabricengine#Topology.add`
- `internal/fabricengine#Topology.Add`

**Create:**
- `internal/fabricengine#resolveAddParent`
- `internal/fabricengine#addParent`
- `internal/fabricengine_test#TestAdd_FromParentPair`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#SourceDirtyPaths`
- `internal/fabricengine#ReadOriginFor`
- `internal/hubforge#AddPair`
- `internal/hubforge#CopyHub`

**Intent:**
D5: `AddOptions` gains `Parent string`, a pair slug of the hub, and `Beam bool`, the beam mark written into the new pair's origin record in the same origin commit (D9 step 2, D12).
Empty `Parent` keeps today's fork from the acting worktree's HEAD and its tracked-dirt refusal (`ErrSourceDirty`).

With `Parent` named, `resolveAddParent` runs among the pre-flight refusals, before any mutation, and returns the parent's worktree location, its checked-out branch and its records branch (`RecordsBranchName` of that branch). It refuses, each naming the slug and the way forward:
- a parent that is not a pair of this hub: no linked warp worktree at `WorktreePath(l, parent)` in `List`; the refusal names `lyx fabric pairs` to list the hub's pairs;
- the prime's own name, naming dropping `--parent` and running the add from the prime to fork from it, since without `--parent` the add forks from the acting worktree's HEAD;
- a parent worktree on a detached HEAD or an unborn branch, naming `lyx fabric checkout <parent branch>` in the parent's worktree, its pair branch `cfg.BranchPrefix + parent`;
- a parent whose checked-out branch is not its own pair branch, `cfg.BranchPrefix + parent`, naming that branch and `lyx fabric checkout <branch>` in the parent's worktree.

With `Parent` named, `Add` checks no tracked dirt at all, neither the parent's nor the acting worktree's, and logs the parent's dirty paths (`SourceDirtyPaths` of the parent location) at Info; the warp branch forks from the parent worktree's HEAD (`git worktree add -b <branch> <path> <parent-head-sha>`), the weft branch forks from the parent's records branch, and the origin record carries the parent's branch as `ParentBranch` and `parent` as `ParentWorktree`.
The acting location keeps every other role it has today (hub, anchor, junction wiring, launchers), so a caller in any worktree that passes the prime's location forks from the named parent.
Bound: what `Parent` lets through is a fork whose base is the parent's last commit, never its uncommitted edits; the refusals above bound it to a live pair of this hub on its own branch.
The refusals name the CLI flag `--parent` (card 6), which is how an operator sets `Parent`.
`AddOptions.Beam` sets `Origin.Beam` in the record `Add` writes; an adopted weft that already carries a record keeps that record unchanged, as today.

`TestAdd_FromParentPair` over a hubforge hub: add pair `beam`, commit a file on it, leave a tracked file dirty in both `beam` and the prime, add `member` with `Parent: "beam"` from the prime's location and assert the member's warp HEAD equals `beam`'s committed HEAD, its records branch forked from `beam`'s records branch, and the origin record names `beam`'s branch and worktree;
a row with `Beam: true` asserts the mark; rows for an unknown parent, the prime's name, a detached parent and a parent switched to another branch each assert the refusal text and that no worktree, branch or record was created;
a row with no `Parent` and a dirty prime still refuses with `ErrSourceDirty`.
The package doc's `Add` section gains the parent fork, its refusals and the dirt rule.

**ImpactSummary:** Add gains one optional parent path; every existing caller passes no Parent and keeps today's behaviour.
