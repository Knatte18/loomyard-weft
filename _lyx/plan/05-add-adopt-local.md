# Card 5 — add-adopt-local

**Edit:**
- `internal/fabricengine#AddOptions`
- `internal/fabricengine#Topology.add`
- `internal/fabricengine#ErrBranchExists.Error`
- `internal/fabricengine_test#TestAdd_ExistingBranchErrorNamesRemedy`

**Create:**
- `plan:internal/fabricengine_test#TestAdd_AdoptLocal` -> `func TestAdd_AdoptLocal(t *testing.T)`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#ReadOriginFor`
- `internal/hubforge#CopyHub`

**Intent:**
D6: `AddOptions.AdoptLocal bool` lets `Add` make a pair over an unmarked local warp branch that already exists, instead of refusing `ErrBranchExists`.
Only `lyx fabric add --adopt-local` (card 6) sets it; batten, adopt and every other caller leave it unset and keep the refusal.
Under `AdoptLocal` the existing branch is checked out into the new worktree (`git worktree add <path> <branch>`) and, like an adopted marked branch, is never deleted by the rollback; the weft side follows today's rules (adopt a live records branch, else fork from the parent's records branch).
Before any mutation it refuses a warp branch checked out in another worktree, naming that worktree and `lyx fabric checkout` to move it off, and likewise a records branch checked out in another records worktree, naming that worktree and, as the way forward that releases the records branch without tearing any pair down, switching that records worktree off it in that worktree, back to its own pair's records branch, or detached with `git switch --detach` when it belongs to no pair, and then re-running `lyx fabric add <slug> --adopt-local`.
The origin record's parent comes from `Parent` or the acting worktree as in card 4, except when the adopted branch is the warp's default branch: then `ParentBranch` and `ParentWorktree` are empty, a root pair, which batten's Land row already refuses to land on its no-parent-branch check.
Bound: only a command naming the flag adopts a local branch, so an unattended run never adopts a stale branch left under a reused slug.
`ErrBranchExists.Error` keeps its text and appends `--adopt-local` as a third way forward (`; or make a pair over it with "lyx fabric add <slug> --adopt-local"`), so the hand-written copies of its opening in battenshed's and battencli's tests stay true, and `TestAdd_ExistingBranchErrorNamesRemedy` asserts the new text, the remedy text being the only place it is pinned.
`TestAdd_AdoptLocal` over a hubforge hub: a local branch `x` with a commit is adopted (pair HEAD equals that commit, origin names the acting worktree's branch); without the flag the same add still refuses `ErrBranchExists`; a branch checked out in another worktree is refused naming that worktree and `lyx fabric checkout`;
the records branch of `x` checked out in another records worktree, with no warp branch checked out elsewhere, is refused naming that records worktree and `git switch --detach` in it;
adopting the default branch records an empty parent branch and worktree: the row first detaches the prime (`git switch --detach` in the prime), so the default branch is checked out nowhere, adds with the slug whose `BranchPrefix + slug` names the default branch from a pair's location, and asserts both parent fields empty.

**ImpactSummary:** One new option on Add and a reworded refusal text; existing callers pass no flag and keep the refusal.
