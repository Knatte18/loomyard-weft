# Card 2 — rollbackAdd deletes the warp branch this Add created

**Create:**
- `internal/fabricengine/add_rollback_branch_integration_test.go`

**Edit:**
- `internal/fabricengine/destroy.go`
- `internal/fabricengine/add.go`
- `internal/fabricengine/remoteleftover.go`
- `internal/fabricengine/add_rollback_adopt_test.go`
- `internal/fabricengine/destructivegaps_integration_test.go`
- `internal/fabricengine/livestate_verbs_test.go`
- `//cmd/lyx/destructiveguard_test.go`

**Prosa:**
- `pattern/PATTERN-fabric-git.md`

**Intent:**
D2: `rollbackAdd` proves ownership of the warp branch by its creation in this `Add` call, not by `ownedManagedBranch`'s naming scheme, so a failed `Add` under the default empty `branch_prefix` no longer strands the bare-slug branch.

In `destroy.go` (`PATTERN-fabric-destruction-chokepoint`):
- A new unexported proof type for a created branch, minted only in `destroy.go` when the `git worktree add -b <branch>` that `createGitWorktree` runs succeeds, the same way `createdToken` is minted; the caller tells the minter the branch name its `-b` created, and the adopted-from-origin `--track -b` form mints it too.
- A new `branchOwnershipKind` with its `ownedXxx` constructor taking that proof, whose predicate accepts exactly the proof's branch name, wired into `resolveBranchOwnership`.
- A new remote-branch dirtiness kind for "pushed by this call", which `checkRemoteBranchRequest` accepts only with a non-empty `leaseSHA`.
- The gate order stays containment, ownership, dirtiness, force.

`cmd/lyx/destructiveguard_test.go` bans the new proof type's composite-literal token outside `destroy.go`, beside `createdToken{`.

`probeWarpLeftover` reports a three-valued outcome to `Add`: the probe did not run (`SkipGit`/`SkipPush`), the branch is absent on origin, or present (adopted, or a leftover at or behind `HEAD`).

In `add.go`, `Add` carries the created-branch proof, the probe outcome, whether step 11's push was attempted, and the commit step 11 pushed (the warp branch's tip read just before the push) to every `rollbackAdd` call.
`rollbackAdd`'s step 5 then:
- deletes the local warp branch through the new ownership kind whenever this `Add` created it, forked or adopted, since an adopted local copy carries only origin's tip;
- deletes the origin warp branch only when the probe ran and found it absent and step 11's push was attempted, through `deleteRemoteBranch` with `leaseSHA` set to the pushed commit, so a branch moved since is refused;
- never touches an origin branch the probe found, any weft branch, or any other ref.
When step 11 never landed the lease finds no ref at that commit, so the deletion is refused; a ref that landed despite the refusals is deleted.
Every refusal or failure is logged with `logger.Warn` naming the branch and returned, since every caller discards `rollbackAdd`'s return; only a deletion that happened is recorded in `rec`, through `deleteRemoteBranch`'s existing success-only entry.

What this lets through, and its bound: the rollback's "no origin branch is ever touched" rule weakens only for the one warp ref the same `Add` call created, pinned by lease to the commit it pushed.
A real non-fast-forward or hook rejection at step 12 therefore deletes the warp branch pushed seconds earlier, when the probe found it absent.

Docs in this commit: `add.go`'s file header and `rollbackAdd`'s doc drop the "left behind" residue and state the origin exception and its bound; the step 5 logger line no longer calls the left-behind branch expected; `pattern/PATTERN-fabric-git.md`'s teardown section gains one line stating the exception.

Coverage, `integration`-tagged on hubforge hubs, with Card 1's push seam on the `Topology` and a hook-declining pre-receive for a real step-12 refusal:
- `add_rollback_branch_integration_test.go`:
  a weft push that keeps failing with `(failed)` stops after the attempt bound and rolls back with no warp branch locally or on origin under the empty `branch_prefix`, and a re-add of the same slug then succeeds;
  a step-11 push that keeps failing with `(failed)` without landing leaves origin without the branch and the deletion refused;
  a step-11 runner that pushes for real and then reports `(failed)` has the landed ref deleted;
  a warp branch moved on origin after step 11's push (a pre-receive on the weft bare that advances the warp bare's branch and declines) is left on origin;
  a pre-existing origin warp branch at or behind `HEAD` that step 11 pushes forward survives on origin while the local copy is deleted;
  an `Add` under `SkipPush` whose weft step fails deletes only the local branch.
- `add_rollback_adopt_test.go`: a rollback whose warp branch was adopted from origin deletes the local copy and leaves origin's branch.
- `destructivegaps_integration_test.go` and `livestate_verbs_test.go`: assertions that the rollback leaves the warp branch behind flip to its absence.

**ImpactSummary:** `Add`, `rollbackAdd` and the destruction gate's branch-ownership and remote-dirtiness sets change; no other verb constructs the new kinds, and the guard keeps the proof unforgeable outside `destroy.go`.

**Commit:** 2: rollbackAdd deletes the warp branch its Add created (fixes #433)
