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
The diff is read at the start of Finalize, before any parent-side mutation.
Landing never merges the weft, so the merge-base is always the task's fork point and every run of Finalize, resumed or not, computes the same diff.
A read failure never halts the landing: Finalize logs a warning, lands as usual, and at Done queues one notice saying the slug's config diff could not be read and naming the two branches, so the hub diffs them by hand.
When the diff is non-empty and Finalize reaches Done, it queues one single-line orch notice through `orchengine.QueueNotice` at the prime's paths, as batten does: the task slug, the changed config files, the merge-base and tip SHAs, and the instruction to re-apply on the parent's `_lyx/config/` each change meant for the parent.
An empty diff queues nothing.
A `QueueNotice` error is a logged warning and never changes the verdict.
`queued=false` because the orch state records no strand is accepted as is, since `QueueNotice` already logs the notice and returns no error; the hub sees a notice only where a strand is recorded.
Bound: Finalize never writes the parent's config and never stops on this output; the notice is the only new output, and a Finalize that reaches Done again after a resume queues it again.
Rationale: per-pair tuning is legitimate (the hub sets a small task's `review_max_bounces` in the pair's `loom.yaml`), so an automatic carry would push per-pair overrides into the parent; the bug is the silence, and the hub is the only party that knows which change was meant for the parent.
The two cases the board cites were `board.yaml`, now HubWide since `board-config-per-worktree`, and PATTERN, now in warp; the remaining exposure is the per-worktree modules.

### D5 — One weft-branch resolver: local, then origin, then none

Decision: fabricengine gets one function that resolves a pair's weft branch for `add`, `reconcile` and `checkout`:

1. local `refs/heads/<slug>-weft` exists: use it;
2. else `origin/<slug>-weft` exists: fetch it and create the local branch tracking it;
3. else: no branch, and the caller forks as today.

A failed origin probe or fetch is an error from the resolver, never a fall-through to step 3.
A weft repo with no `origin` remote configured skips step 2, as `cleanup` already treats it; only an unreachable or failing origin is an error.
The branch name comes from `WeftBranchName`.
`add` passes its skip-git and skip-push options, and the resolver skips step 2 when either is set, which leaves today's fork behavior for that offline opt-out; `reconcile` and `checkout` take no such option and always consult origin.
Bound: step 2 only creates a local branch whose content is already on origin, so nothing is lost; the new failure mode is an unreachable origin, which now stops the verb instead of forking.
Only `add`'s skip opt-out forks past an origin weft; `reconcile` and `checkout` have no opt-out and the scope adds none, so for them an unreachable origin stops the pair or verb with an error naming origin, and the way forward is restoring access, including for `checkout` to a branch with no weft anywhere, which forked offline before.
Rationale: forking a fresh weft beside a live origin weft is the bug; `clone` already adopts this way for the prime.

### D6 — `add`'s live-pair rule

Decision: `lyx fabric add <slug>` treats the pair as live when the weft branch exists locally, or exists on origin with no `archive/<slug>/*` tag covering its tip; archive tags are consulted only when there is no local weft branch.
With `SkipGit` or `SkipPush` no origin is consulted: the pair is live only by a local weft branch, and the warp branch forks from HEAD as today.
The live-pair decision, the divergence refusal and the origin probes run before the first creation, so a refusal leaves nothing behind.
For a live pair:

- the weft comes from D5 (local, or origin-tracked);
- if the local weft branch is a strict ancestor of the origin one, it is fast-forwarded to the origin tip before adoption; a true divergence keeps today's `leftoverDivergedAdoptedWeft` refusal;
- if `origin/<slug>` exists, the warp branch is created tracking it, whatever its relation to HEAD, replacing the `leftoverDivergedWarp` refusal for this case;
- if `origin/<slug>` is absent, the warp branch forks from HEAD as today.

For a pair that is not live (no local weft branch, and an origin weft that is absent or archive-covered), behavior is unchanged: an archived remote weft is replaced, and a diverged remote warp is still refused.
The refusal for an uncovered remote weft (`leftoverUnarchivedWeft` and its message) is retired, since that weft now makes the pair live.
A local warp branch that already exists is still `ErrBranchExists`.
A live pair's adopted weft that already carries an origin record keeps it: Add neither rewrites nor commits it, so a stacked task moved between machines keeps its recorded parent.
An adopted weft with no record gets one written and committed as on today's local-adopt path, naming the acting worktree as parent; the weft is then pushed.
Rollback deletes only local branches this Add created (an origin-tracked branch included), never an origin branch; a pre-existing local weft branch that was fast-forwarded is not rewound, since it only moved to a tip origin already holds.
Bound: the replaced refusal now lets a remote warp branch with commits HEAD lacks into a new local branch, only when the weft side proves a live pair: a surviving local weft branch (`Remove` deletes the pair's local branches, so it is the moved-back case) or an unarchived origin weft.
A removed pair's leftovers with an archived weft and no local weft still hit today's refusals; an unarchived weft whose archive step failed is the accepted case in R3, and nothing on origin is replaced in it.
Rationale: moving a running task to another machine is the case; tying warp adoption to a live weft keeps a removed pair's stale warp branch from being resurrected.

### D7 — `reconcile` and `checkout` consult origin before forking

Decision: `reconcileMissingWeft` calls D5 before its raw-warp and unmanaged branches; an origin weft is adopted through the `weft_recreated` path, so junctions, portal and launchers are wired as with a local branch.
`reconcile` adopts an origin weft whether or not an archive tag covers it; this covers the raw-warp and the unmanaged (managed warp worktree, no weft sibling) cases alike, and both end as `weft_recreated` with the wiring repair, where the raw path alone ended as a dormant `raw_adopted` with a fresh weft before.
For the unmanaged case the justification is the managed warp worktree on that branch, which proves a pair; the raw case rests on the weaker bound below.
Bound: that bypass of the archive guard only creates a local branch from origin and pushes or replaces nothing, and only for a slug whose warp worktree already exists locally on that branch, so no removed pair is revived without a local warp worktree; `add`'s replace-archived rule applies only where `add` itself creates the pair, so the two never decide the same slug.
A raw warp worktree proves no live pair, so for it the bound is weaker: a raw worktree on a branch reusing a removed pair's slug adopts that pair's archived weft and has junctions wired into it, which is accepted because the content was already on origin, nothing is pushed or replaced, and the hub can remove the pair again.
A resolver error is a per-pair `Error`, and no dormant weft is forked.
`Checkout`'s `switchOrForkWeft` calls D5 before forking; an origin-tracked branch is switched to, and the rollback treats it as a created branch (deleted on rollback, origin untouched).
`checkout` adopts an origin weft whether or not an archive tag covers it, as `reconcile` does; its bound is that it creates no pair and only switches an existing weft worktree onto a local branch tracking content origin already holds, so a checkout of a branch reusing a removed pair's slug replaces the worktree's own per-worktree config and run records with the archived weft's, and nothing on origin changes.
That replacement is bounded by the operator having named the branch and by D4's notice reporting config changes at landing.
Rationale: the board names both paths as forking past an existing origin weft.

### D8 — Regression tests drive the real CLI wiring

Decision: each fix's regression test runs the `lyx fabric` cobra commands (or, for D4, the loom Finalize wiring) on a `hubforge` fixture built through `fabriccli.CloneAndWire` with a bare origin, in the `integration` tier, under `gitkit.HermeticGitEnv()`; fakes may cover extra rows but never replace the wiring test.
Rationale: the board requires CLI-wiring regression tests; `PATTERN-hubforge-fixtures` and `PATTERN-test-speed` set the fixture and tier.

### D9 — Plan test targets by file path

Decision: cards name production code as their main targets and name test files by path, never by test-function glyph.
Rationale: a test-suite prune in flight can rename or delete the fabric test functions this task extends; a file path stays valid, and only the functions inside it are re-validated.

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

- A1: The notice channel, not a Stuck gate, is the right surface for D4, since a stop could only be cleared by a new verb, which the scope forbids; a failed diff read uses the same channel, since the notice protects no correctness of the landing; a notice that cannot be queued, the failed-read one included, is logged, never a halt, and reaches the hub only where an orch strand is recorded.
- A2: The prime's orch notice queue is the D4 recipient even for a stacked task whose parent is another pair, matching batten's existing notices.
- A3: For a live pair, adopting `origin/<slug>` even when it is an ancestor of HEAD is right; the board's "never creates a new local branch in its place" outweighs starting from a newer base.
- A4: `reconcile` and `checkout` may need network now for a pair or branch with no local weft branch; failing that pair or verb is preferred to forking.
- A5: The local-behind-origin fast-forward in D6 is part of adoption, not an extension: without it, moving a task back to the machine it came from refuses.

## Open risks

- R1: D4's diff includes the hub's own per-pair tuning, so tuned tasks raise a notice the hub then ignores; accepted as one line per landing.
- R2: A Finalize resumed after its landing completed computes the same diff again, for a squash and a non-squash landing alike, so the hub may get the notice twice; a notice is never lost to a resume.
- R3: An unarchived remote weft from a removed pair whose archive step failed now makes `add` adopt it, and its remote warp branch with it, instead of refusing; nothing is lost and nothing on origin is replaced, but stale run records and the stale warp branch come back, and a plain `lyx fabric remove` archives the pair again.
- R4: The ReadOriginFor error changes `loomcli`'s reviewer resolution from "no reviewer" to an error for the gap case; reconcile fixes the gap, and D1 stops it arising.

## Acceptance criteria

- After `lyx fabric reconcile` recreates a weft worktree from an existing local weft branch, `.weft/` exists, and `hubgeom.ResolveParent` (through the reed geometry) yields the recorded parent's orch name.
- After `lyx fabric add`, on both the fork and adopt paths, the new weft worktree has `.weft/`.
- A pair whose weft worktree has the origin record but no `.weft/`: `ResolveParent` returns an error naming the missing lock directory and `lyx fabric reconcile`; running `lyx fabric reconcile` then creates `.weft/` and the parent resolves; the pair's reconcile `Detail` names the repair and its action is the one it would otherwise have had.
- A task whose weft changed a per-worktree config file (for example `loom.yaml`) since its fork point: Finalize queues one orch notice naming the slug and the file; a task with no such change queues none; a change to a HubWide module's worktree copy is not reported; the parent's config is unchanged in all cases.
- A Finalize whose config diff read fails still lands and queues one notice saying the diff could not be read, with the parent's config unchanged; a notice that cannot be queued, whether `QueueNotice` returns an error or `queued=false` for want of an orch strand, leaves the verdict unchanged.
- A Finalize resumed after its landing completed computes the same diff and queues the notice again, whether the landing squashed or not.
- `lyx fabric add <slug>` on a hub with no local branches, where origin holds a live pair's `<slug>` and `<slug>-weft` (with commits the hub's HEAD lacks and run records on the weft): the pair is built on both origin tips, both local branches track origin, the run records are present, the weft worktree has `.weft/`, and the add's push is a fast-forward.
- `lyx fabric add <slug>` where origin holds only a live pair's `<slug>-weft`: the weft is adopted tracking origin and the warp branch forks from HEAD.
- `lyx fabric add <slug>` with a skip option and an origin weft present: both branches fork as today and origin is not consulted.
- `lyx fabric add <slug>` with no local weft branch, where origin's weft is covered by an archive tag and origin's warp diverges: today's refusal is unchanged.
- `lyx fabric add <slug>` with a local weft branch, where origin's weft is archive-tagged and its warp diverges: the pair is live and the warp branch is adopted tracking origin.
- `lyx fabric add <slug>` where the local weft branch is behind origin's: it is fast-forwarded and adopted; a true divergence still refuses; an Add that then fails leaves the fast-forwarded branch at origin's tip.
- `lyx fabric reconcile` on a raw warp worktree checked out on `<slug>` with only `origin/<slug>-weft` present: the action is `weft_recreated`, the weft tracks origin, `.weft/` exists, and junctions, portal and launchers are wired; with origin unreachable, the pair reports an error and no weft branch is forked; with an archive tag covering the origin weft, the weft is still adopted.
- `lyx fabric reconcile` on a managed warp worktree (its `_lyx` wired) with no weft sibling and only `origin/<slug>-weft` present: the action is `weft_recreated`, the weft tracks origin, and the wiring is repaired.
- `lyx fabric checkout <branch>` where only `origin/<branch>-weft` exists: the weft worktree switches to a local branch tracking it, also when an archive tag covers that origin weft; a later failure rolls the local branch back and leaves origin untouched.
- `lyx fabric add <slug>` adopting a live pair's weft that already carries an origin record: the record keeps its recorded parent, and no new record commit is made.
- `lyx fabric checkout <branch>` with origin unreachable and no local weft branch: the verb fails naming origin, both sides stay on their original branches, and no weft branch is forked.
- `lyx fabric reconcile` and `lyx fabric checkout` on a hub whose weft repo has no `origin` remote fork as today.
- An Add that fails after creating an origin-tracked branch rolls back the local branch and leaves every origin branch as it was.
