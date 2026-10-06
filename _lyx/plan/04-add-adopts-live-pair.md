# Card 4 — add-adopts-live-pair

**Edit:**
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#Topology.rollbackAdd`
- `internal/fabricengine#probeWeftLeftover`
- `internal/fabricengine#probeWarpLeftover`
- `internal/fabricengine#weftLeftover`
- `internal/fabricengine#ErrRemoteLeftover`
- `internal/fabricengine#leftoverKind`
- `internal/fabricengine#KindRepoAdvanced`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/add_remoteleftover_integration_test.go`
- `internal/fabricengine/origin_integration_test.go`
- `internal/fabriccli/addleftover_integration_test.go`
- `internal/fabriccli/adoptremote_integration_test.go`

**Delete:**
- `internal/fabricengine#leftoverUnarchivedWeft`

**Uses:**
- `plan:internal/fabricengine#resolveWeftBranch`
- `internal/fabricengine#remoteHeadTip`
- `internal/fabricengine#remoteArchiveTags`
- `internal/fabricengine#isAncestorOrEqual`
- `internal/fabricengine#leftoverDivergedAdoptedWeft`
- `internal/fabricengine#leftoverDivergedWarp`
- `internal/fabricengine#WriteOrigin`
- `internal/fabricengine#removeWeftWorktree`

**Intent:**
Fixes `fabric-add-adopt-remote` for `add` (D5's add caller, D6): `lyx fabric add <slug>` adopts a live pair's branches from origin instead of forking or refusing.

Live-pair rule.
The pair is live when the weft branch exists locally, or exists on origin with no `archive/<slug>/*` tag covering its tip; archive tags are consulted only when there is no local weft branch.
With `SkipGit` or `SkipPush`, no origin is consulted: the pair is live only by a local weft branch, and the warp branch forks from HEAD as today.
The live decision, the divergence refusals and every origin probe run in Add's pre-flight, before the first creation, so a refusal leaves nothing behind (the probes stay read-only apart from `FETCH_HEAD`).

Pre-flight, `probeWeftLeftover` (its `weftLeftover` answer gains `live bool` and `fastForwardTo string`):
- local weft branch present: `live=true`; origin absent, or origin tip equal to or an ancestor of the local tip, needs nothing; otherwise fetch the origin tip into `FETCH_HEAD` and, when the local tip is a strict ancestor of it, set `fastForwardTo` to the origin tip; a true divergence keeps the `leftoverDivergedAdoptedWeft` refusal.
- no local weft branch: origin absent is not live; an archive-covered origin tip is not live and keeps today's replace answer (`tip`, `tag`); an uncovered origin tip is `live=true` (adopt from origin).
- Retire `leftoverUnarchivedWeft` and its message branch in `ErrRemoteLeftover.Error`, and update the `leftoverKind` and `ErrRemoteLeftover` comments: an uncovered origin weft now makes the pair live.

`probeWarpLeftover` takes the live answer and returns the origin warp tip to adopt: for a live pair, a present `origin/<slug>` is adopted whatever its relation to HEAD (replacing the `leftoverDivergedWarp` refusal for that case); for a pair that is not live, today's ancestor-of-HEAD check and refusal stand unchanged.

Creation, in Add's existing order:
- Warp: with an adopt tip, fetch `refs/heads/<warp>:refs/remotes/origin/<warp>` in the acting warp worktree, then create the warp worktree with `worktree add --track -b <warp> <path> origin/<warp>` through `createGitWorktree`; otherwise fork from HEAD as today.
  A local warp branch that already exists is still `ErrBranchExists`.
- Weft fast-forward: when `fastForwardTo` is set, advance the local weft branch with `git fetch --no-tags origin refs/heads/<weft>:refs/heads/<weft>` in the weft repo root (git refuses a non-fast-forward), and record `KindRepoAdvanced` with the weft repo root as Target and the branch and new tip as Detail; extend `KindRepoAdvanced`'s comment to name this advance.
- Weft: for a live pair call `resolveWeftBranch(rec, l, weftBranch, !opts.SkipGit && !opts.SkipPush)`; an existing local branch is adopted as today, a branch it created from origin is adopted the same way (`containedWorktreeAdd` without `-b`); a pair that is not live forks through `createWeftWorktreeDroppingRuns` as today.
  A resolver error rolls back through `rollbackAdd`.
- Origin record: when the adopted weft worktree already carries the origin record (`OriginRecordPathFor` exists on disk), Add neither rewrites nor commits it, so a task moved between machines keeps its recorded parent; card 1's adopt-path `ensureWeftLockDirAt` still creates `.weft/`.
  An adopted weft with no record gets one written and committed naming the acting worktree as parent, as today; a forked weft is unchanged.
- Step 12's archived-leftover replacement runs only for the not-live replace answer, as today; the weft push then sets upstream (a no-op push for an unchanged adopted branch).

Rollback: `rollbackAdd` deletes only local branches this Add created; its `weftBranchAdopted` argument now means "the local weft branch existed before this Add", so a weft branch the resolver created from origin is deleted on rollback while origin is never touched, and a pre-existing local weft branch that was fast-forwarded is not rewound.
The warp branch keeps today's gate behaviour (left behind under an empty `branch_prefix`).
Update the rollback and Add doc comments and the add.go header.

Bound: the replaced warp refusal now lets a remote warp branch with commits HEAD lacks into a new local branch, only when the weft side proves a live pair: a surviving local weft branch (the moved-back case, since `Remove` deletes the pair's local branches) or an unarchived origin weft.
A removed pair's leftovers with an archived weft and no local weft still hit today's refusals; an unarchived weft whose archive step failed is the accepted R3 case, where nothing on origin is replaced.
Nothing here deletes or force-pushes an origin branch, and no new message names a denied command (`PATTERN-no-denied-recovery`).

Docs: rewrite `internal/fabricengine/doc.go`'s `Add` re-add paragraph for the live-pair rule, the adopt-from-origin paths and the retired unarchived-weft refusal.

Tests:
- `internal/fabricengine/add_remoteleftover_integration_test.go`: `TestAdd_UnarchivedWeftLeftoverRefusedAtPreflight` becomes the origin-weft-only adopt row (weft adopted tracking origin, carries the pushed commit, warp forks from HEAD); `TestAdd_AdoptedWeftDivergedRefusedAtPreflight` becomes the local-behind-origin fast-forward row, plus a true-divergence row (a local commit origin lacks) that still refuses, and a row where Add then fails (rollback hook or injected failure) leaving the fast-forwarded local branch at origin's tip; add a row where a local weft branch and an archive-tagged origin weft coexist with a diverged origin warp, and the warp is adopted tracking origin; `TestAdd_WarpLeftoverRefusedAtPreflight` keeps covering archived weft plus diverged warp, refusal unchanged; `TestAdd_SkipPushSkipsLeftoverProbes` asserts both branches fork and origin is not consulted.
  An Add that fails after the resolver created the origin-tracked weft branch deletes that local branch and leaves every origin branch unchanged.
- `internal/fabricengine/origin_integration_test.go`: adopting a live pair's weft that already carries an origin record keeps its recorded parent and adds no record commit; update any local-adopt assertion that expected the record rewritten.
- `internal/fabriccli/addleftover_integration_test.go`: the bare pre-flight error envelope is now reached through a still-refused case (archive-covered weft, no local weft, diverged origin warp), since an uncovered origin weft no longer refuses.
- `internal/fabriccli/adoptremote_integration_test.go`, through `lyx fabric add`: origin holds a live pair's `<slug>` and `<slug>-weft` (pushed from scratch clones, with commits the hub's HEAD lacks and a run record under the weft's run-records root) and the hub has neither locally: the pair is built on both origin tips, both local branches track origin, the run record is present, `.weft/` exists, and origin's tips are unchanged after the add's push; origin holding only `<slug>-weft`: weft adopted tracking origin, warp forked from HEAD.

**ImpactSummary:** Add's pre-flight probes, creation steps, origin-record step and rollback; ErrRemoteLeftover loses its unarchived-weft kind; fabriccli runAdd's envelope handling is unchanged.

**Commit:** 4: fabric: add adopts a live pair's branches from origin
