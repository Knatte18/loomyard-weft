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
- `internal/fabricengine/doc.go`
- `internal/battencli/wire.go`
- `internal/battencli/lifecycle_integration_test.go`
- `internal/landingshed/publish.go`
- `//cmd/lyx/destructiveguard_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `pattern/PATTERN-fabric-git.md`
- `docs/overview.md`

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
`internal/fabricengine/doc.go`'s sentence "A rollback deletes only the local branches this `Add` created ... and never restores the deleted remote branch" states the same exception: the warp branch this `Add` created is deleted locally, and on origin only under the probe-absent, step-11-attempted and lease conditions, while the replaced weft branch is still never restored.
The `doc.go` paragraph on the token-carrying ownership kinds also covers the created-branch proof, stating that, like `createdToken`, its unforgeability rests on the composite-literal ban in `cmd/lyx/destructiveguard_test.go` and not on the type system, and its heading stops stating a count of kinds.

Statements that say a rolled-back create keeps its branch, which this card makes false, change with it:
- `internal/battencli/wire.go`: `createRefusal`'s comment and its user-visible message drop the clause "a rolled-back create keeps the branch it made", leaving a torn-down pair and a create rolled back by an older binary as the sources of a leftover branch (`ErrBranchExists`' remedy covers those).
  `wire_test.go` and `lifecycle_integration_test.go` assert none of the removed clause, and the doc comment of `stepCreateRow_LeftoverBranchIsRewordedForPrime` in `lifecycle_integration_test.go` drops "(or a rolled-back create)" the same way.
- `internal/landingshed/publish.go`: the comment that keeps a failed `Add` push as Stuck "because Add's rollback keeps the branch it made" says instead that `Add` retries a bare `(failed)` refusal itself and rolls back what it created, so a push that still fails is left Stuck for a human.
- `docs/overview.md`: the sentence on a create refused on a leftover branch drops "and a rolled-back create keeps the branch it made".

The new test file adds top-level test functions to `internal/fabricengine`, whose row in `cmd/lyx/testdata/test-budget.yaml` has no headroom, so this card raises that row to the package's top-level test count after the card, counting the test it deletes below (`PATTERN-test-speed`).

Coverage, `integration`-tagged on hubforge hubs, with Card 1's push seam on the `Topology` and a hook-declining pre-receive for a real step-12 refusal.
Existing tests that pin the old behaviour are reworked rather than duplicated (`PATTERN-test-economy`):
- `add_rollback_branch_integration_test.go` (new):
  a weft push that keeps failing with `(failed)` stops after the attempt bound and rolls back with no warp branch on origin under the empty `branch_prefix`, and a re-add of the same slug then succeeds, which needs the local branch gone too;
  a step-11 push that keeps failing with `(failed)` without landing leaves origin without the branch and the deletion refused;
  a step-11 runner that pushes for real and then reports `(failed)` has the landed ref deleted;
  a pre-existing origin warp branch at or behind `HEAD` that step 11 pushes forward survives on origin while the local copy is deleted;
  an `Add` under `SkipPush` whose step-11 runner pushes for real and then reports an error deletes only the local branch and leaves origin's branch, since `SkipPush` skips the probe and nothing after step 11 can fail under it.
- `add_rollback_adopt_test.go`:
  `TestAddRollback_WarpBranchLeftBehindUnderEmptyPrefix` flips to assert the bare-slug branch is deleted under the default empty `branch_prefix`, and is renamed to say so;
  `TestAddRollback_RefusedWarpBranchDeletionLogsWarn` is reworked to the lease refusal, replacing its gate-refusal assertions: a warp branch moved on origin after step 11's push (a pre-receive on the weft bare that advances the warp bare's branch and declines) stays on origin, and the WARN line names the branch and the refused lease;
  a rollback whose warp branch was adopted from origin deletes the local copy and leaves origin's branch, as a case beside `TestAddRollback_LiveWeftFromOrigin`'s.
- `destructivegaps_integration_test.go`: `TestBranchOwnership_RefusalHoldsAtOtherDeletionSites` is deleted, since it exists to show the ownership refusal at the rollback site, which D2 removes for a branch this `Add` created, and the local-only `SkipPush` rollback it would now pin is covered by the new file's `SkipPush` case; `TestAddRollback_RefusesJunctionRemovalOutsideItsWorktree`'s comment that a prefix makes the branch deletable is rewritten.
- `livestate_verbs_test.go`: the `Add` expectation that the rollback leaves the warp branch behind flips to its absence, the comment explaining the empty-prefix gap is rewritten, and the `addCase` doc comment stops citing the deleted `TestBranchOwnership_RefusalHoldsAtOtherDeletionSites` as the source of its broken-origin injection.

**ImpactSummary:** `Add`, `rollbackAdd` and the destruction gate's branch-ownership and remote-dirtiness sets change; no other verb constructs the new kinds, the guard keeps the proof unforgeable outside `destroy.go`, and the other non-test changes are two comments, one refusal message's wording in `battencli`, the package doc and the test budget row.

**Commit:** 2: rollbackAdd deletes the warp branch its Add created (fixes #433)
