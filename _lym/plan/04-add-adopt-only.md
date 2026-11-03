# Card 4 — add-adopt-only

**Create:**
- `plan:internal/fabricengine#ErrPairNotLive` -> `var ErrPairNotLive error`
- `plan:internal/fabricengine#ErrNotLive` -> `type ErrNotLive struct`
- `plan:internal/fabricengine#ErrNotLive.Error` -> `func (e *ErrNotLive) Error() string`
- `plan:internal/fabricengine#ErrNotLive.Is` -> `func (e *ErrNotLive) Is(target error) bool`
- `plan:internal/fabricengine#ErrRecordsWorktreeNotPair` -> `type ErrRecordsWorktreeNotPair struct`
- `plan:internal/fabricengine#ErrRecordsWorktreeNotPair.Error` -> `func (e *ErrRecordsWorktreeNotPair) Error() string`
- `plan:internal/fabricengine#ErrAdoptLocalCheckedOut` -> `type ErrAdoptLocalCheckedOut struct`
- `plan:internal/fabricengine#ErrAdoptLocalCheckedOut.Error` -> `func (e *ErrAdoptLocalCheckedOut) Error() string`
- `plan:internal/fabricengine#ErrTaskPathTaken` -> `type ErrTaskPathTaken struct`
- `plan:internal/fabricengine#ErrTaskPathTaken.Error` -> `func (e *ErrTaskPathTaken) Error() string`
- `plan:internal/fabricengine#PruneRegistrations` -> `func PruneRegistrations(l *lymcwd.Location) error`
- `plan:internal/fabricengine_test#TestAdd_AdoptOnly` -> `func TestAdd_AdoptOnly(t *testing.T)`

**Edit:**
- `internal/fabricengine#AddOptions`
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#Topology.add`
- `internal/fabricengine#refuseAdoptLocalCheckedOut` -> `func refuseAdoptLocalCheckedOut(l *lymcwd.Location, slug, warpBranch, weftBranch string, weftExists bool, reusedWeft string) error`
- `internal/fabricengine#Topology.rollbackAdd` -> `func (t *Topology) rollbackAdd(rec *Mutations, l *lymcwd.Location, slug, warpBranch, weftBranch, target string, weftBranchAdopted, weftReused bool, warpTok createdToken, warp *addWarpBranch) error`
- `internal/fabricengine/doc.go`
- `internal/fabriccli/doc.go`

**Uses:**
- `internal/fabricengine#probeWeftLeftover`
- `internal/fabricengine#probeWarpLeftover`
- `internal/fabricengine#recordAdoptedWeftAnchor`
- `internal/fabricengine#List`
- `plan:internal/gitrepo#Repo.DropRemoteTracking`
- `internal/fabricengine_test#requireNothingCreated`
- `internal/fabricengine_test#removedPair`
- `internal/fabricengine_test#pushNewPair`

**Intent:** Decision record D1 step 3 and D2's closed set of permanent answers, on fabric's side: an adopt-only `Add` that re-makes a live pair and never forks one.

`AddOptions` gains `AdoptOnly bool`. Under it, `Add` forks nothing.
Its not-live answer is `*ErrNotLive` with fields `Branch` and `Records bool` (true for the weft branch), whose `Is` matches the sentinel `ErrPairNotLive`, so a caller tests the answer with `errors.Is` and reads which branch with `errors.As`:
- Right after the weft probe, a weft that is not live (`weftOld.live` false: absent locally and on origin, or origin's tip covered by an `archive/<slug>/*` tag) returns `*ErrNotLive` naming the weft branch, before any mutation.
- After the warp probe, a task branch neither local nor on origin returns `*ErrNotLive` naming the task branch, before any mutation.
- The warp probe runs for an adopted local warp branch too (it is skipped there without `AdoptOnly`); its `onOrigin` answer serves only card 3's tracking-ref drop, and the adopted local branch is never fetched or replaced from origin.
- It skips every check that guards only a fork it never makes: the source-dirt check (`ErrSourceDirty`), the read of the acting worktree's HEAD with its detached-or-unborn refusal, and the fork point's `refuseEmptyOwnMarker` and `moveRecordStop`; it resolves no parent, since the adopted weft's origin record names it, so a parent pair removed since cannot refuse it.
  An adopted weft with no origin record is written none under `AdoptOnly` (the record is never invented), and the add goes on.
- `ErrRemoteLeftover` under `AdoptOnly` can only be a local weft branch diverging from origin's, since the warp leftover refusal is reached only for a pair that is not live.
- When `RecordsWorktreePath(l, slug)` stands and `List(RecordsRepoRoot)` shows it registered, not prunable, on the pair's weft branch, `Add` reuses it: it skips the `weft worktree directory already exists` refusal and the weft `containedWorktreeAdd`, takes the anchor that worktree records through `recordAdoptedWeftAnchor`, makes only the task worktree over the local or origin task branch and wires the pair as its adopt path does.
  `refuseAdoptLocalCheckedOut` gains `reusedWeft`, the reused worktree's path or empty, and does not count that worktree as another records worktree holding the branch.
  `rollbackAdd` gains `weftReused` and, when set, leaves that worktree and its branch in place, as it leaves an adopted branch; every one of its callers is inside `Topology.add`.
- A records worktree directory that stands without being such a registered worktree returns `*ErrRecordsWorktreeNotPair` (fields `Path`), replacing today's plain `weft worktree directory already exists` error for every caller, text unchanged.

Typed errors replacing today's plain ones, texts unchanged, so a caller can test D2's closed set with `errors.As`:
- `refuseAdoptLocalCheckedOut`'s two refusals become `*ErrAdoptLocalCheckedOut` with fields `Branch`, `Path` and `Records bool`.
- The `worktree directory %q already exists` refusal at the task worktree's location becomes `*ErrTaskPathTaken` with field `Path`.

`PruneRegistrations` runs `git worktree prune` in the code repository (the acting worktree) and in the records repository (`RecordsRepoRoot`), dropping every stale registration, as `removeSibling` already prunes before a branch deletion; it is batten's D1 step 2, a general fabric verb any caller may take.

Bound: `AdoptOnly` re-creates only a live pair: never one whose weft is gone locally and archived or absent on origin, never one whose task branch is gone locally and on origin, never over a stray records directory or task path.
It is set only together with `AdoptLocal`, so the rule that only `--adopt-local` sets `AdoptLocal` becomes: the flag sets it, and otherwise only batten's D1 resume does, under `AdoptOnly`.
Rewrite that rule in the `AddOptions.AdoptLocal` comment, in `Topology.Add`'s doc comment (which also gains the adopt-only answers, the reused records worktree and the typed refusals), in `internal/fabricengine/doc.go`'s `AdoptLocal` paragraph and in `internal/fabriccli/doc.go`'s `--adopt-local` line, and add an `AdoptOnly` paragraph to `internal/fabricengine/doc.go`.

Test `TestAdd_AdoptOnly`, `tier1`, one hub per row, every row passing `AdoptLocal` and `AdoptOnly`:
- Weft branch on origin with no archive tag, task branch on origin, both worktrees and local branches gone: the pair is re-made, the task worktree on the task branch, the records worktree on the weft branch with its run records.
- The slug's unmarked local warp branch kept, its worktree directory deleted by hand so its registration is stale: after `PruneRegistrations` the pair is re-made over the local branch.
- A beam project's pair (`AddOptions.Beam` at creation, subpath anchor via `Anchor`) whose task worktree is gone: re-made at the anchor its weft records.
- The prime with tracked dirt: re-made.
- Weft live, task branch gone locally and on origin: `errors.Is(err, ErrPairNotLive)`, `requireNothingCreated`.
- Weft archived by a removal (`removedPair`): `ErrPairNotLive`, nothing created.
- Weft branch absent locally and on origin: `*ErrNotLive` with `Records` true, nothing created.
- The slug's unmarked local warp branch kept behind a newer copy on origin: the pair is re-made over the local branch at its own tip, and no `refs/remotes/origin/<warp>` is written.
- An adopted weft branch whose tree carries no origin record: the pair is re-made and the weft branch gains no origin-record commit.
- Local weft diverging from origin's: `*ErrRemoteLeftover`.
- Records worktree standing registered on the weft branch, task worktree alone deleted: only the task worktree is made and wired to the standing records worktree; a failure injected after it (`SetAddFailAfterWarpHookForTest`) leaves the records worktree in place.
- A plain directory at the records worktree path: `*ErrRecordsWorktreeNotPair`.
- A file at the task worktree path: `*ErrTaskPathTaken`.
- The task branch checked out in another worktree: `*ErrAdoptLocalCheckedOut` with `Records` false.
- An adopted local warp branch origin lacks, under a stale `refs/remotes/origin/<branch>`: the ref is gone after `Add`.

**ImpactSummary:** One new option and three typed errors on `Add`, whose existing callers (fabriccli add, batten's Worktree-Create row, reconcile) set no `AdoptOnly` and see the same texts; `refuseAdoptLocalCheckedOut` and `rollbackAdd` are re-signed with every caller inside `Topology.add`.

**Commit:** `4: Add re-adopts a live pair under an adopt-only option and types its refusals`
