# Decision record: pair-lifecycle

## Goal

Removing a pair ends its reed session first, finishes a half-removed pair, preserves every run record in the archive tag, and leaves no landed branch behind locally or, with `--remote`, on origin.
Starting a run never adopts a driver strand that is about to remove itself.

## Scope

In:

- One pair-teardown composite that both `lyx fabric remove` and batten's Worktree-Teardown row call: wait for the loom driver to go quiet, end the pair's reed session, then remove the pair.
- `fabricengine.Remove` finishes a half-removed pair instead of answering "worktree not found".
- `Remove` commits the pair's pending run records before the archive tag, so a late stop report reaches the archive.
- `remove --remote` deletes the pair's warp branch on origin when its work is landed.
- `fabric cleanup --remote` finds and deletes leftover warp branches on origin that belong to no live pair, carry no open PR, and hold no unlanded work.
- reed marks a strand that has scheduled its own removal as retiring, and `lyx loom start` replaces a retiring driver strand instead of adopting it.
- The `main-weft` item: closed as already resolved and by design, with one doc sentence.
- Docs and `CONSTRAINTS.md` updates listed under Constraints.

Out:

- Teaching reed's watchdog to reap a session whose worktree path exists but is no longer a registered worktree.
- Deleting a stray directory at a pair path that is not a registered worktree (reported only, see D5).
- Advancing `main-weft` on landing.
- Any change to `fabric prune`'s behavior.
- Batten's driver-exit grace in its InnerRun done arm stays as it is.

## Decisions

### D1 — One pair-teardown composite in a new package

A new package (working name `internal/pairteardown`) owns the teardown sequence: quiescence wait (D2), a read-only refusal probe (D3), reed session shutdown (D4), then `fabricengine.Remove`.
`fabriccli`'s `remove` verb and `battencli`'s Worktree-Teardown deps both call it; batten keeps one producer that sequences shutdown before removal, per the Batten Bookend Invariant.
battencli's own half-removed special-casing (the `PairSiblingRemnant` refusal and the `RemovePairBranch` re-entry path) goes, since `Remove` now finishes those states (D5); exported fabric helpers left with no caller are deleted.

Rationale: `reedengine` imports `fabricengine`, so fabric cannot end a session itself without an import cycle, and two call sites must not drift.
The package sits outside the Fabric Vocabulary owner set and says "pair" and "task worktree", never warp or weft.

### D2 — Wait for the loom driver to go quiet, bounded

Before anything is torn down, the composite waits until the pair's loom driver is quiet: the run lock is free, and the loom driver strand (`loomengine.IsDriverStrand`) is absent, dead, retiring (D7), or parked (park marker present).
It polls on a fixed interval up to a fixed bound (2 minutes for `fabric remove`).
On timeout the policy is the caller's:

- `fabric remove` refuses, leaving the pair and its session untouched, naming the live driver, the pair's `lyx reed attach` command and "retry once the driver has written its report, or end it with `lyx reed down` in the pair".
- batten passes a zero wait and end-and-report: it has already waited its `driver_exit_grace_s` in the InnerRun done arm, so a driver still live is ended by D4 and recorded as an abandoned session, as today.

Bound: the wait never kills a non-quiet driver on the `fabric remove` path; only batten's existing grace-then-end behavior ends one, and only after its configured grace.
A parked driver is ended without refusal: its park command committed its records, and removing the pair already ends the run it would resume.
No new CLI flag.

Rationale: the driver commits its run records before it retires or parks, so waiting for either is what makes its stop report reach the archive (a commit that failed is caught by D6), and an unbounded wait would hang the hub's land step.

### D3 — Refusals run before the session is ended

`Remove`'s refusals that do not depend on teardown (slug validation, prime refusal, merge in progress in either direction, warp-side dirtiness without `--force`, and sibling-side dirtiness outside the scoped pathspec without `--force`, see D6) are exposed as a read-only probe the composite runs after D2 and before D4.
A call the probe refuses leaves the session, strands and pair untouched.
`Remove` still runs the same checks itself; a refusal only `Remove` raises comes from a change made between probe and `Remove`, and leaves the session already ended.

Rationale: an operator who mistypes `remove` on a pair with uncommitted edits must not lose their interactive strands to a call that then refuses.

### D4 — End the pair's reed session before any worktree is removed

The composite ends the pair's reed session with `reedengine.Engine.Down`, whether or not the task worktree is still on disk; when it is gone, the session is addressed by the name reed derives from hub and slug.
Down's existing behavior also tears down the hub's tmux server and its socket when this was the last session.

Rationale: a live strand writing into a removed pair recreated `<hub>/<slug>/.lyx/...` and `_lyx/shed/<slug>/drive-reports/`, which also defeated the watchdog's root-gone reap rule; with the session ended first, no writer remains.

### D5 — `Remove` finishes a half-removed pair

When the task worktree is absent, `Remove` no longer refuses with "worktree not found".
It performs whatever teardown remains, in the normal order: pending-record commit (D6), archive, portal and launchers, sibling worktree, both branches locally, and with `--remote` both branches on origin (D8).
A path at the pair location that is not a registered linked worktree is reported in a result field and never deleted; the rest of the teardown proceeds.
When nothing of the pair remains (no worktree on either side, no branch locally or on origin, no portal or launcher entry), `Remove` returns a typed not-found error: the CLI reports it as an error, and batten treats it as the done post-condition.
The not-found error names a stray path at the pair location when one is present.
`RemoveResult` reports which steps this call performed.

Bound: the finish path deletes only what the full path would have deleted, through the same destruction gates; it adds no new deletion target.

Rationale: a re-run must converge on the post-condition instead of stranding the operator in a state only hand git can clear.

### D6 — `Remove` commits pending run records before archiving

After the refusals and before the archive, `Remove` commits the sibling worktree's uncommitted changes through the same scoped pathspec `lyx fabric sync` uses, in both the full and the finish path, with or without `--force`.
The no-force refusal on sibling-side dirtiness inside the scoped pathspec goes; the warp-side dirtiness refusal stays.
Sibling-side changes outside the scoped pathspec are refused without `--force`, as today, and that check runs with the other refusals, before the commit, so a refusal precedes every mutation.
With the task worktree gone, the commit's warp-SHA trailer uses the local warp branch tip when present, else the last recorded `Warp-SHA` on the sibling branch.

Bound: this replaces a refusal whose own remedy was "run `lyx fabric sync`" with doing exactly that sync; it commits only lyx-owned record paths, on the pair's own branch, and never touches the task worktree.
`force` still answers dirtiness only.

Rationale: a stop report written late must reach the archive tag, and after a half-removal nothing else can ever commit it.

### D7 — reed marks a self-removing strand as retiring; `loom start` replaces it

`lyx reed remove --name <n> --detach` records the strand as retiring in reed's state before it spawns the detached remover, and reed's strand status exposes the flag.
The detached remover removes by guid, and a guid no longer present is a clean no-op.
`lyx loom start` treats a live driver strand marked retiring as gone: it removes that strand synchronously by guid, then spawns a fresh driver.
The same flag counts as quiet in D2.

Bound: the mark records only that some caller of `reed remove --detach` requested the strand's removal, whether the driver's teardown command or an operator by hand; start and D2 only complete that request.
The mark does not prove the records were committed: the teardown command runs the removal even when its commit failed, and records left uncommitted stay in the sibling worktree for D6's commit or the next run's commit.
A detached-remover spawn failure clears the mark, so a strand is never left marked retiring with no remover behind it.
A live driver strand without the mark keeps today's handling (working, parked, or the not-parked refusal at a hand-back).

Rationale: the retiring state is reed's own lifecycle bookkeeping, not caller data, so it fits reed's dumb-carrier rule; a fresh strand gets a new guid, so the late remover cannot delete it.

### D8 — `remove --remote` deletes the warp branch on origin when landed

With `--remote`, after the local warp-branch step, `Remove` deletes the pair's warp branch on origin through the destruction gate when its remote tip's work is landed: reachable from another ref, or merging it into the recorded parent's tip changes nothing (the existing `checkUnlandedWork` rule applied to the fetched remote tip).
For a remote tip, "another ref" excludes every copy of the same branch: its local branch and every remote-tracking ref of that name.
Without the exclusion the branch's own `refs/remotes/origin/<branch>` always holds the tip, and since the local step already counts that ref, an unlanded pushed branch would lose both copies.
The deletion is leased on the observed remote tip.
A branch that fails the gate is kept with a reason on the result, and `Remove` still succeeds.

Bound: deletes only `BranchPrefix + slug` for the pair being removed, only with `--remote`, only with landed work and only at the leased tip.

Rationale: squash landings mean git never sees the branch as merged, and the merge-tree check is the existing squash-proof landed test.

### D9 — `fabric cleanup --remote` sweeps leftover warp branches on origin

With `--remote`, `Cleanup` also enumerates warp branches on origin and classifies each.
A branch is fabric-managed when the weft origin holds an `archive/<slug>/*` tag or a `<branch>-weft` branch for it.
It is a deletion candidate when it is fabric-managed, is not origin's default branch, is not checked out in any hub worktree, is not the head of an open PR, and its tip's work is landed on origin's default branch (D8's rule, with D8's exclusion of the branch's own copies).
Open PR heads are fetched in `fabriccli` through `internal/githubclient` and passed to the engine as a protected set; when GitHub cannot be reached, warp-branch deletion is skipped for the whole call with a reason, and the weft-branch sweep proceeds.
Without `--apply` it reports only; with `--apply` it deletes candidates leased on their observed tip, and reports every non-candidate with its reason.

Bound: an unmanaged branch, a live pair's branch, an open PR's head and any branch with unlanded work are never deleted; `--force` stays reserved and answers none of these.

Rationale: `fabricengine` stays free of network calls, and an unreachable GitHub fails toward keeping branches.

### D10 — `main-weft` needs no code change

`main-weft` is pushed already.
The fabricengine package doc gains one sentence stating that `main-weft` does not advance on landing by design.

Rationale: weft content is per-branch and never a merge participant (Durable-vs-Ephemeral State Invariant), and a landed pair's records live in its archive tag, so advancing it would contradict an existing invariant.

## Constraints

- `fabricengine` gains no import of `reedengine`, `loomengine` or `githubclient`.
- Every destructive step stays in `internal/fabricengine/destroy.go` behind its gates (Fabric Destruction Chokepoint Invariant); every new remote deletion is leased.
- Every mutating result keeps its mutation record (Mutation Record Invariant); new steps append only after they changed state.
- The archive tag still precedes every branch deletion, and `force` never skips it (Fabric Git Invariant).
- The new composite package and every non-owner file stay inside the Fabric Vocabulary Invariant.
- Tests that spawn tmux follow the Tmux Test Isolation Invariant; git tests follow the Hermetic Git Test Environment Invariant; spawning tests carry the integration build tag (Test Tier Purity Invariant).
- `CONSTRAINTS.md`: a new invariant stating that every pair teardown goes through the composite, which ends the pair's session before any worktree is removed; the Fabric Git Invariant's teardown bullet gains the pending-record commit before the archive.
- Docs in the same commit: fabricengine, reedengine, loomcli start, battenshed and the new package's docs; the fabric CLI help for `remove` and `cleanup`; the module doc in `manifest/designs/`; `docs/overview.md`'s module table for the new package.

## Auto-mode assumptions

- "Sockets" in the board title means the hub's tmux server socket; D4 covers it through Down's existing last-session server teardown, so no separate socket work is planned.
- The 2-minute quiescence bound for `fabric remove` is a constant, not config; a refusal on timeout is cheap to retry.
- The `driver-restart-race` scenario as written (hand-back then self-removal) predates parking; the reachable window today is a driver at done or at a busy refusal running its teardown command, which D7 closes.
- A stray non-worktree directory at a pair path is rare once D4 lands, so reporting it is enough.

## Open risks

- Two concurrent teardowns of the same pair (an orch `fabric remove` while batten tears down) are not serialized by the composite; batten holds the prime lock, `fabric remove` does not.
- D9's landed test conflicts when origin's default branch later edited the same lines a squash-landed branch changed, so such a branch is reported as unlanded and kept; that errs toward keeping.

## Acceptance criteria

- `fabric remove` on a pair with a live, working driver strand waits, then refuses at the bound with the attach command; the pair, session and strands are untouched.
- `fabric remove` on a pair whose driver retires during the wait proceeds once it is retiring, and the archive tag contains the driver's committed stop report.
- `fabric remove` refused for warp-side dirtiness leaves the pair's reed session and strands alive.
- `fabric remove` refused for sibling-side changes outside the scoped pathspec leaves the pair's reed session, strands and sibling branch untouched.
- After `fabric remove`, the pair's reed session is gone, and no path is recreated under `<hub>/<slug>`.
- A stop report left uncommitted in the sibling worktree is committed and contained in the archive tag after `remove` (full path and finish path).
- Half-removed states each converge on one re-run of `remove`: task worktree gone with sibling worktree present and dirty; both worktrees gone with branches left locally; both gone with branches left only on origin (`--remote`).
- `remove` on a slug with nothing left returns the typed not-found error; batten's teardown treats it as done.
- A non-worktree directory at the pair path is reported and left on disk while the rest of the teardown completes; when it is all that remains, the not-found error names it.
- `remove --remote` deletes the warp branch on origin after a squash landing, keeps it with a reason when its work is unlanded, and does not delete it when the remote tip moved after it was observed.
- `remove --remote` on a pair whose unlanded warp branch is pushed keeps the origin copy with a reason.
- `cleanup --remote` dry-run lists a landed managed leftover as a candidate, and lists with reasons: an unmanaged branch, a live pair's branch, an open PR's head, an unlanded branch and origin's default branch; `--apply` deletes only the candidate.
- `cleanup --remote` with GitHub unreachable skips every warp-branch deletion with a reason and still sweeps weft branches.
- `reed remove --name --detach` marks the strand retiring before the remover runs; the remover is a no-op when its guid is already gone.
- `loom start` with a live retiring driver strand spawns a fresh driver, and the old strand's detached remover leaves the new strand alive.
- `loom start` with a live, unmarked driver strand behaves as today in the working, parked and not-parked cases.
- batten's teardown ends a driver still live after its grace and records it as an abandoned session, as today, through the composite.

## Notes for the plan writer

- Teardown today: `internal/fabricengine/remove.go` (`Remove`, `refuseDirtyWeftWorktree`), `internal/fabricengine/cleanup.go` (`Cleanup`, `RemovePairBranch`), remote deletion executors in `internal/fabricengine/destroy.go`, `checkUnlandedWork` for the landed rule.
- batten's current two-dep teardown and half-removed handling: `internal/battencli/wire.go` (Teardown deps), `internal/battenshed/teardown.go`.
- Driver strand handling in start: `internal/loomcli/start.go` (`runDriverSpawnAndWait`), `internal/loomcli/bootstrap.go` (`resolveDriverStrandAction`), the teardown command in `internal/loomcli/driverprompt.go`.
- Self-removal: `internal/reedcli/remove.go` (`spawnDetachedRemove`, `waitParentExit`).
- Session geometry: `internal/hubgeom` (`ReedGeometry`); watchdog's root-gone rule in `internal/reedcli/watchdog.go`.
- PR lookup precedent: `internal/landingshed/pullrequest.go`.
