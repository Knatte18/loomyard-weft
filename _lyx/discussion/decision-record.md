# Decision record — bugfix-fabric

## Goal

Fix the three bugs the board entry `bugfix-fabric` reports, each with a regression test that drives the real CLI wiring:

- `weft-lock-dir-drops-parent`: a weft worktree created or recreated without its `.weft/` lock directory makes its strands lose `LYX_PARENT` silently.
- `config-reaches-parent`: a config change a task makes in its own `_lyx/config/` dies with the task's weft branch at landing, and nothing says so.
- `fabric-add-adopt-remote`: `lyx fabric add`, `reconcile` and `checkout` ignore a pair's existing branches on origin and fork or refuse instead of adopting them.

## Scope

In:

- Every fabricengine path that creates or recreates a weft worktree creates `.weft/`.
- `lyx fabric reconcile` creates a missing `.weft/` in an otherwise healthy pair.
- `fabricengine.ReadOriginFor` turns "record present, lock directory missing" into an error naming the gap; `hubgeom.ResolveParent` propagates it.
- Finalize detects a task's changes to its per-worktree config files and reports them to the hub's orch notice queue.
- One fabricengine resolver for a pair's weft branch (local, then origin, then none), used by `add`, `reconcile` and `checkout`.
- `lyx fabric add` adopts a live pair's warp branch from origin instead of refusing it as a diverged leftover.
- Docs in the touched packages' `doc.go` for the changed observable behavior.

Out:

- New verbs, flags, config keys or envelope keys, except the `fabric-add-adopt-remote` behavior the board explicitly puts in scope.
- Carrying config changes into the parent automatically.
- `clone`'s existing origin adoption for the prime, which stays as it is.
- Warp-side `checkout` of a branch that exists only on origin (git's own switch behavior stays).

## Decisions

### D1 — Every weft worktree creation creates `.weft/`

Decision: each fabricengine helper that runs a weft `git worktree add` (the fork path, the drop-runs fork path, `add`'s adopt path, `reconcile`'s `adoptWeftWorktree`, and the new origin-adopt path from D7) calls `ensureWeftLockDirAt` on the new weft worktree right after the worktree exists, inside the same mutation, before it returns success.
A failure to create it fails that verb through its existing rollback or per-pair error path.
Rationale: the lock directory is what `ReadOriginFor` and every weft writer rely on; creating it at birth closes the window the 2026-10-05 move hit, and `ensureWeftLockDirAt` already seeds the weft excludes, so no new raw write appears (`PATTERN-fabric-write-containment` is untouched because `.weft/` is in a weft worktree, not under `_launchers` or `_portals`).

### D2 — Reconcile heals a missing `.weft/`

Decision: `lyx fabric reconcile` creates `.weft/` in any pair whose weft worktree exists but lacks it, as part of its existing per-pair repair, and reports it in that pair's `Detail`.
It adds no new `ReconcileAction` value; a pair whose only repair is this keeps the action it would otherwise have, with the detail appended.
Rationale: D3's error must name a way forward, and reconcile is the verb that already repairs pair wiring; reusing the existing action set keeps the envelope unchanged (no new envelope key or action vocabulary).

### D3 — A present origin record without `.weft/` is an error

Decision: `ReadOriginFor` keeps answering "no record" when the weft worktree or the record file is absent.
When the record exists and the `.weft/` directory does not, it returns an error that names the weft worktree path, says the lock directory is missing, and names `lyx fabric reconcile` as the way forward.
It still creates nothing.
`hubgeom.ResolveParent` returns that error wrapped; `parentNameOrEmpty` already logs it as a warning naming the consumer, and `loomcli`'s reviewer resolution already returns it.
`Remove` ignores `ReadOriginFor` errors today and keeps doing so: its fallback (empty parent branch, stricter reachability check) is the safe direction.
Bound: the only answer that changes is the "record present, lock missing" case, which today returns a silent "no record"; no caller gains a write.

### D4 — Finalize reports a task's config changes, never carries them

Decision: Finalize computes the task's changes to its per-worktree config files: the diff restricted to `_lyx/config/` between the merge-base of the task's weft branch and its parent's weft branch, and the task's weft tip.
Only files of config modules not marked `HubWide` in `configreg` count; a HubWide module's worktree copy is never read.
The computation is a fabricengine function (git on weft stays in fabricengine per `PATTERN-fabric-git`), handed to landingshed through its deps as a told function (landingshed derives no path per `PATTERN-told-geometry`, and names no fabric side per `PATTERN-fabric-vocabulary`).
The diff is read before the parent-side merge; a read failure is Stuck with the parent untouched, naming a rerun of the run as the way forward.
When the diff is non-empty and Finalize reaches Done (on the merge path and on the already-landed path alike), it queues one single-line orch notice through `orchengine.QueueNotice` at the prime's paths, as batten does: the task slug, the changed config files, the fork-point and tip SHAs, and the instruction to re-apply on the parent's `_lyx/config/` each change meant for the parent.
An empty diff queues nothing.
Bound: Finalize never writes the parent's config; the notice is the only new output, and a resumed Finalize may queue it twice.
Rationale: per-pair tuning is legitimate (the hub sets a small task's `review_max_bounces` in the pair's `loom.yaml`), so an automatic carry would push per-pair overrides into the parent; the bug is the silence, and the hub is the only party that knows which change was meant for the parent.
The two cases the board cites were `board.yaml`, now HubWide since `board-config-per-worktree`, and PATTERN, now in warp; the remaining exposure is the per-worktree modules.

### D5 — One weft-branch resolver: local, then origin, then none

Decision: fabricengine gets one function that resolves a pair's weft branch for `add`, `reconcile` and `checkout`:

1. local `refs/heads/<slug>-weft` exists: use it;
2. else `origin/<slug>-weft` exists: fetch it and create the local branch tracking it;
3. else: no branch, and the caller forks as today.

A failed origin probe or fetch is an error from the resolver, never a fall-through to step 3.
The branch name comes from `WeftBranchName`; the resolver honors the verbs' existing skip-git and skip-push options by skipping step 2 when set.
Bound: step 2 only creates a local branch whose content is already on origin, so nothing is lost; the new failure mode is an unreachable origin, which now stops the verb instead of forking.
Rationale: forking a fresh weft beside a live origin weft is the bug; `clone` already adopts this way for the prime.

### D6 — `add`'s live-pair rule

Decision: `lyx fabric add <slug>` treats the pair as live when the weft branch exists locally, or exists on origin with no `archive/<slug>/*` tag covering its tip.
For a live pair:

- the weft comes from D5 (local, or origin-tracked);
- if the local weft branch is a strict ancestor of the origin one, it is fast-forwarded to the origin tip before adoption; a true divergence keeps today's `leftoverDivergedAdoptedWeft` refusal;
- if `origin/<slug>` exists, the warp branch is created tracking it, whatever its relation to HEAD, replacing the `leftoverDivergedWarp` refusal for this case;
- if `origin/<slug>` is absent, the warp branch forks from HEAD as today.

For a pair that is not live, behavior is unchanged: an archived remote weft is replaced, an uncovered one cannot occur (it would make the pair live), and a diverged remote warp is still refused.
A local warp branch that already exists is still `ErrBranchExists`.
Add's origin record is written and committed on the adopted weft as on today's local-adopt path, naming the acting worktree as parent, and pushed.
Rollback deletes only local branches this Add created (an origin-tracked branch included), never an origin branch.
Bound: the replaced refusal now lets a remote warp branch with commits HEAD lacks into a new local branch, only when the weft side proves a live pair; a removed pair's leftovers (archived weft) still hit today's refusals.
Rationale: moving a running task to another machine is the case; tying warp adoption to a live weft keeps a removed pair's stale warp branch from being resurrected.

### D7 — `reconcile` and `checkout` consult origin before forking

Decision: `reconcileMissingWeft` calls D5 before its raw-warp and unmanaged branches; an origin weft is adopted through the `weft_recreated` path, so junctions, portal and launchers are wired as with a local branch.
`reconcile` adopts an origin weft whether or not an archive tag covers it, because a live warp worktree on that branch is present.
A resolver error is a per-pair `Error`, and no dormant weft is forked.
`Checkout`'s `switchOrForkWeft` calls D5 before forking; an origin-tracked branch is switched to, and the rollback treats it as a created branch (deleted on rollback, origin untouched).
Rationale: the board names both paths as forking past an existing origin weft.

### D8 — Regression tests drive the real CLI wiring

Decision: each fix's regression test runs the `lyx fabric` cobra commands (or, for D4, the loom Finalize wiring) on a `hubforge` fixture built through `fabriccli.CloneAndWire` with a bare origin, in the `integration` tier, under `gitkit.HermeticGitEnv()`; fakes may cover extra rows but never replace the wiring test.
Rationale: the board requires CLI-wiring regression tests; `PATTERN-hubforge-fixtures` and `PATTERN-test-speed` set the fixture and tier.

## Constraints

- `PATTERN-fabric-git`: all git on warp or weft, including the D4 diff and the D5 fetch, stays in `internal/fabricengine` through `gitexec.Run` (`PATTERN-gitexec-checked-call`); no new `//gitexec:raw` site unless it is a bool predicate, as `weftBranchExists` is.
- `PATTERN-gitrepo-client-boundary`: not touched; fabricengine already uses `gitexec` for remote work.
- `PATTERN-told-geometry` and `PATTERN-fabric-vocabulary`: landingshed receives the D4 diff function and the notice queue through its deps; its code, strings and comments name neither fabric side.
- `PATTERN-orch-pane-single-writer`: Finalize only queues a notice through `orchengine.QueueNotice`; notices are single-line.
- `PATTERN-mutation-record`: every new branch creation, fetch-created branch and fast-forward in `add`, `reconcile` and `checkout` appends to the verb's `*Mutations` record; no new envelope key.
- `PATTERN-fabric-destruction-chokepoint`: rollback of an origin-tracked branch goes through the existing gate (`deleteBranch`), never a raw `branch -D`.
- `PATTERN-no-denied-recovery`: no new error message names `git push --force`, `git reset --hard` or `rm -rf`.
- `PATTERN-never-force-add`: `.weft/` stays out of the index through the weft excludes `ensureWeftLockDirAt` seeds.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-hubforge-fixtures`: regression tests extend existing tests where one covers the path, sit in `integration`, and use hubforge.
- `PATTERN-comment-line-breaks`: semantic line breaks in changed Go comments.
- Board body: bugfix only; no new verbs, config keys or features beyond `fabric-add-adopt-remote`.

## Auto-mode assumptions

- A1: The notice channel, not a Stuck gate, is the right surface for D4, since a stop could only be cleared by a new verb, which the scope forbids.
- A2: The prime's orch notice queue is the D4 recipient even for a stacked task whose parent is another pair, matching batten's existing notices.
- A3: For a live pair, adopting `origin/<slug>` even when it is an ancestor of HEAD is right; the board's "never creates a new local branch in its place" outweighs starting from a newer base.
- A4: `reconcile` may need network now for a pair with no local weft branch; failing that pair is preferred to forking.
- A5: The local-behind-origin fast-forward in D6 is part of adoption, not an extension: without it, moving a task back to the machine it came from refuses.

## Open risks

- R1: D4's diff includes the hub's own per-pair tuning, so tuned tasks raise a notice the hub then ignores; accepted as one line per landing.
- R2: A crash between Finalize's push and the notice loses the notice on the merge path unless the resume reaches the already-landed path, which re-queues it (D4 covers both paths).
- R3: An unarchived remote weft from a removed pair whose archive step failed now makes `add` adopt it instead of refusing; nothing is lost, but stale run records come back with it.
- R4: The ReadOriginFor error changes `loomcli`'s reviewer resolution from "no reviewer" to an error for the gap case; reconcile fixes the gap, and D1 stops it arising.

## Acceptance criteria

- After `lyx fabric reconcile` recreates a weft worktree from an existing local weft branch, `.weft/` exists, and `hubgeom.ResolveParent` (through the reed geometry) yields the recorded parent's orch name.
- After `lyx fabric add`, on both the fork and adopt paths, the new weft worktree has `.weft/`.
- A pair whose weft worktree has the origin record but no `.weft/`: `ResolveParent` returns an error naming the missing lock directory and `lyx fabric reconcile`; running `lyx fabric reconcile` then creates `.weft/` and the parent resolves.
- A task whose weft changed a per-worktree config file (for example `loom.yaml`) since its fork point: Finalize queues one orch notice naming the slug and the file; a task with no such change queues none; a change to a HubWide module's worktree copy is not reported; the parent's config is unchanged in all cases.
- `lyx fabric add <slug>` on a hub with no local branches, where origin holds a live pair's `<slug>` and `<slug>-weft` (with commits the hub's HEAD lacks and run records on the weft): the pair is built on both origin tips, both local branches track origin, the run records are present, and the add's push is a fast-forward.
- `lyx fabric add <slug>` where origin's weft is covered by an archive tag and origin's warp diverges: today's refusal is unchanged.
- `lyx fabric add <slug>` where the local weft branch is behind origin's: it is fast-forwarded and adopted; a true divergence still refuses.
- `lyx fabric reconcile` on a raw warp worktree checked out on `<slug>` with only `origin/<slug>-weft` present: the action is `weft_recreated`, the weft tracks origin, and junctions, portal and launchers are wired; with origin unreachable, the pair reports an error and no weft branch is forked.
- `lyx fabric checkout <branch>` where only `origin/<branch>-weft` exists: the weft worktree switches to a local branch tracking it; a later failure rolls the local branch back and leaves origin untouched.
- An Add that fails after creating an origin-tracked branch rolls back the local branch and leaves every origin branch as it was.

## Notes for the plan writer

- Weft creation sites to start from: `createWeftWorktree` and `createWeftWorktreeDroppingRuns` (`weftwiring.go`), `Add`'s adopt branch (`add.go`), `adoptWeftWorktree` (`reconcile.go`).
- Origin probes to reuse: `remoteHeadTip`, `remoteArchiveTags`, `isAncestorOrEqual` (`remoteleftover.go`); `suffixWeftPrimaryBranch` (`clone.go`) shows the tracking checkout.
- The leftover probes in `Add` run before the first mutation; the live-pair decision belongs there, so a refusal still comes before any creation.
- Finalize's parent-side merge is landingshed's merge critical section; the diff read goes before it.
