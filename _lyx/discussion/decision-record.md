# run-pushes — decision record

## Goal

A task pair's branch is on the remote after every loom transition, whatever made its commits, and a loom status push recovers by itself from a rejection that is not a real divergence (#418).
The push is Go's job: no agent prompt is relied on to push.

## Scope

In:

- Loom's per-transition status seam pushes the task branch together with the run records, in one serialized, rebase-free push of both sides.
- `lyx loom commit-records` pushes both sides the same way.
- `lyx fabric push` and `lyx fabric sync` push both sides, as `lyx fabric commit`'s detached push already does.
- The anchored push from Go serializes with fabric's detached push child under the existing absorbing push lock.
- A rejected push retries once when the remote tip is already contained in the local branch; a real divergence still warns and rewrites nothing.
- A new PATTERN entry for the invariant, the package docs that describe today's weft-only push, and tests.

Out:

- Batten's status seam: it runs in the prime, whose code branch is the parent branch the operator pushes; it keeps pushing the records side only, though `PushAnchored` gains the lock and the retry.
- Any change to agent prompts, including Burler's "Never push": agents keep committing with `git commit`, and Go pushes at the next transition.
- Auto-resolving a real divergence (rebase, merge or force push) on either side.
- Publish and Finalize pushes, which already push the task branch.

## Decisions

1. **One both-sides anchored push in `fabricengine`.**
   A new vocabulary-neutral entry beside `PushAnchored` (name chosen at Plan, e.g. `PushPairAnchored`) takes a `*lyxcwd.Location`, resolves both worktrees itself and pushes the code side, then the records side, each through `gitrepo.PushRebaseFree`, under the absorbing push lock `CoalescePushBothAt` takes (`fabric.push.lock` under the records worktree's `.weft/`).
   It takes the lock wait as an input, bounded or unbounded (Decisions 3 and 5).
   It attempts the records-side push even when the code-side push failed.
   Every push failure it returns names the failed side and that side's worktree path and wraps the side's own error with `%w`, a rejection that stays after the retry of Decision 4 included, so `errors.Is(err, gitrepo.ErrPushRejected)` holds for one rejected side and for both.
   Both failed sides return `errors.Join` of the two named errors.
   A lock-wait expiry (Decision 3) returns an error that names the push lock and no side, since neither side was attempted.
   `PushAnchored`, which pushes the records side alone, keeps returning `gitrepo.ErrPushRejected` unwrapped.
   It records each advanced side in its `MutationRecord`.
   Rationale: loomcli may not name the sides (`PATTERN-fabric-vocabulary`), all git stays in `fabricengine` (`PATTERN-fabric-git`), and rebase-free keeps agents' SHAs and the correspondence index intact.
   Records-side push regardless of the code-side outcome keeps a rejected or detached code branch from also stranding the run's status.
2. **Loom's status seam and `commit-records` use it.**
   `loomCommitStatusDeps`'s `Push` calls the new entry instead of `PushAnchored`.
   The statuscommit core's push-warns disposition is unchanged: every push error, the rejection included, warns and the run continues; `commit-records` keeps its error envelope on a push error.
   The warning and the envelope carry the entry's side-named error text, so a one-side failure shows which side failed and a both-sides failure shows both.
   The envelope's fixed text no longer claims the records were not pushed, since the records side may have been pushed while the code side failed: it says the commit landed locally and the push failed, and names no side itself, because `loomcli` may not (`PATTERN-fabric-vocabulary`).
   Only the appended entry error names a side and its worktree path, so the envelope never asserts which side is unpushed beyond what that error says.
   A lock-wait expiry (Decision 3) takes the other-failure wording below, with the entry's no-side error appended.
   Way forward in that message and in its `contracts/specs/refusal-spec.md` row, which splits in two as the Publish push-rejected rows do, both rows of class transient (the verb halts without mutating state and the message names the retry):
   for a rejection that stays after the retry, merge the remote branch into the local branch in the worktree the error names (or clear what a remote rule objects to), then re-run `lyx fabric push` or `lyx loom commit-records`;
   for any other failure: `lyx fabric push` or a re-run of `lyx loom commit-records`.
   `loomcli` picks the row with `errors.Is(err, gitrepo.ErrPushRejected)` on the entry's error, or with a side-neutral predicate `fabricengine` exports; when one side was rejected and the other failed otherwise, the rejection row applies, and its re-run step covers the other failure.
   Neither names a command the agents' settings deny.
   Rationale: the transition seam is the one place Go pushes after every step, so the code branch lags the remote by at most one step.
3. **`PushAnchored` takes the absorbing push lock too.**
   Batten's seam keeps calling it, records side only.
   Its doc comment's "taking no lock is deliberate" reasoning is replaced: the lock is the weft-side `fabric.push.lock`, not `PushCoalesced`'s repo-root lock, and contending with the detached push child is the point.
   Bound: the status-side acquisition, in `PushAnchored` and in the new entry as the seam and `commit-records` call it, waits for the lock a bounded, injectable time and on expiry returns an error the seam warns about, so the run continues and the next transition catches up; the detached child keeps `CoalescePushBothAt`'s blocking acquire, and so does `lyx fabric push` normal mode (Decision 5).
   The bound covers only a waiter.
   A holder runs both sides, each with a possible fetch and retry push, and `gitexec` sets no command timeout, so a holder's length depends on the network and is unbounded; a status push that gets the lock and hangs in git stalls its own transition as a lock-free push does today.
   What the lock adds is that such a hung status push now also blocks the detached children behind it until git returns, which is accepted.
   Rationale: the most likely cause of #418 is the lock-free status push racing a detached `lyx fabric push` child on the same records branch, where git reports the losing ref update as rejected.
4. **One retry on a rejection that is not a divergence.**
   On `ErrPushRejected` for a side, the entry fetches that side's upstream; when the fetched remote tip is an ancestor of, or equal to, local HEAD, it pushes once more; otherwise it returns the rejection.
   Bound: one fetch and one extra push per side per call, never a rebase, merge or force; a remote holding commits the local branch lacks is still left alone and warned about, since another machine advancing the branch is the operator's decision.
   The retry lives in the new entry and in `PushAnchored`, not in `gitrepo.PushRebaseFree`, whose callers keep its plain contract.
   Rationale: it absorbs a race with any pusher outside the lock (an agent's own `git push`, a transient remote ref-lock failure) without touching history.
5. **`lyx fabric push` and `lyx fabric sync` push both sides.**
   `push`'s normal mode calls the new entry of Decision 1 instead of `Fabric.PushWeft`, so a rejection that stays after the retry is an error envelope, never a swallowed warning.
   It asks the entry for an unbounded lock wait, as the repo-root lock of `PushCoalesced` it replaces waited: it is an operator command off the run's transition path, and after a landing commit it must wait out the detached child that `Fabric.Commit` just spawned rather than fail beside it.
   The detached child's own `push` bypass keeps `CoalescePushBothAt`.
   `sync` passes the code-side path to the detached child as `Fabric.Commit` already does.
   `lyx fabric push` therefore no longer runs `PushCoalesced`'s `pull --rebase` on a rejection; Decision 4's bound applies instead, and a real divergence is left alone and reported.
   Bound: both verbs push unconditionally, not only when the call landed a commit as `lyx fabric commit` does, so in the prime they publish any unpushed commits of the parent branch even when the call committed nothing, including through the `sync` that `lyx config` runs after an edit.
   The push is a plain, rebase-free push of commits already made on the checked-out branch, never a force, and `lyx fabric commit` already publishes the same commits whenever it lands something.
   The prime cannot be told apart in the layout, so a parent branch that is behind, diverged from or protected on its remote is rejected there too: `lyx fabric push` still pushes the records side, then returns the error envelope naming the code side and its worktree, and rewrites nothing local.
   The way forward is the rejection row of Decision 2; `lyx fabric sync` is unaffected, since its detached child's own bypass keeps `CoalescePushBothAt`, which warns on a rejection.
   Rationale: the three verbs agree, and `lyx fabric push` is the way forward `commit-records` names after a failed push, so it must carry the task branch.
6. **New PATTERN entry under "Fabric and git".**
   `PATTERN-push-both-sides` — Pushing a run's records side from Go: in a task pair the code branch goes with it, rebase-free, under fabric's absorbing push lock, and pushing is never left to an agent; the prime's per-transition push stays records-only, since its code branch is the operator's parent branch.
   Enforced by review; it lands in the same commit as the code.
   Rationale: the operator's direction that the remote must not rest on agents is cross-cutting and outlives this task.

## Constraints

- `PATTERN-fabric-git`, `PATTERN-gitrepo-client-boundary`, `PATTERN-gitexec-checked-call`: all git through `fabricengine`/`gitrepo`; the fetch for the retry goes through an existing or new `gitrepo` remote call, never raw git elsewhere.
- `PATTERN-fabric-vocabulary`: loomcli, battencli and statuscommit name no side; the new export's name carries neither side.
- `PATTERN-mutation-record`: the new push result embeds `MutationRecord`, appending a push only after a side observably advanced.
- `PATTERN-no-denied-recovery`: no refusal or warning names a form on the pattern's denied list.
- `PATTERN-refusal-way-forward`: the `commit-records` push-failure message and its `contracts/specs/refusal-spec.md` row change together with the code into the two rows of Decision 2, and a test reaches each.
- `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-hubforge-fixtures`: push tests that need a real remote are `integration`-tagged on hubforge fixtures; the seam's dispositions stay covered by stub closures.
- `PATTERN-test-economy`: extend the existing `PushAnchored`, `CoalescePushBothAt`, status-seam and `commit-records` tests before adding files.
- `WEFT_SKIP_GIT` / `WEFT_SKIP_PUSH` (`SyncOptions.SkipGit`/`SkipPush`) skip both sides with no lock taken, as today.
- Docs land in the same commit: `fabricengine`'s doc comments for the anchored pushes, `statuscommit`'s and `loomcli`'s seam comments, `fabriccli`'s `push`/`sync` long help, and the PATTERN entry.

## Auto-mode assumptions

- The #418 cause in Decision 3 is inferred from the code; the drive report was not inspected.
- "Wherever the weft is pushed after a step" means loom's transition seam and `commit-records`; mid-step agent commits may stay local until the step's next transition.
- Batten's per-transition seam in the prime keeps the parent branch out of its pushes; the `push` and `sync` verbs may publish it, as Decision 5 states.

## Open risks

- An agent that amends or rebases an already-pushed commit makes the code side diverge from its remote; the push then warns every transition and Publish's push is rejected until the run's own `lyx webster reset` or the operator resolves it.
- A hung push holder, a detached child or a status push, holds the absorbing lock for as long as git stays stalled, since `gitexec` sets no command timeout; status pushes waiting behind it warn after the wait bound of Decision 3 and the run continues, but the code branch stays behind until a push gets the lock, and detached children and `lyx fabric push` wait behind it without limit.
  A status push that is the holder stalls its own transition, as a lock-free status push does today.
- In the prime, a parent branch that is behind, diverged from or protected on its remote makes `lyx fabric push` return an error envelope after the records side was pushed (Decision 5); the operator merges the remote branch, as the rejection row of Decision 2 says.
- A code-side push from a detached HEAD (mid-rebase) fails and warns each transition until the branch is back.

## Acceptance criteria

- A loom transition after an agent's raw `git commit` on the task branch leaves the remote task branch at local HEAD, along with the records branch.
- `lyx loom commit-records` pushes both sides; a rejected code-side push yields its error envelope, and the records side is still pushed.
- When the two sides fail with different error kinds (code side rejected, records side a network error), the returned error identifies both sides and satisfies `errors.Is(err, gitrepo.ErrPushRejected)`; the status seam warns with both and `commit-records` reports both in its envelope.
- When only one side fails, the returned error names that side and its worktree and satisfies `errors.Is(err, gitrepo.ErrPushRejected)` when that side was rejected; the `commit-records` envelope carries that error and, with the code side failed and the records side pushed, does not claim the records were unpushed.
- `commit-records` reaches its rejection message and its transient message each, and neither names a denied command.
- `commit-records` against a lock held past the wait bound returns the transient envelope with the no-side lock error appended.
- `lyx fabric push` and `lyx fabric sync` advance the remote task branch as well as the records branch.
- `lyx fabric push` against a real divergence returns an error envelope and rewrites neither branch; with only the code side diverged, as for a prime's behind parent branch, the records side is still pushed and the envelope names the code side.
- A rejection whose remote tip is an ancestor of local HEAD is retried once and succeeds, with no warning logged.
- A real divergence (the remote holds a commit the local branch lacks) is not retried past the one fetch, warns at the status seam, and leaves both local branches' SHAs unchanged.
- A status push and a detached `lyx fabric push` child on the same pair serialize on the absorbing lock.
- A status push against a lock held past the wait bound warns and the run continues.
- `lyx fabric push` against a lock held by a detached child waits for it and then pushes both sides.
- `WEFT_SKIP_PUSH=1` pushes neither side and takes no lock.
- Batten's status seam still pushes only the records side.
- The status seam's dispositions (mid-merge skip, commit hard-error, push warns) are unchanged.

## Notes for the plan writer

- `CoalescePushBothAt`'s `pushRebaseFreeLogged` swallows a rejection as a warning; the new entry must surface it so `commit-records` can report it.
- `webstercli`'s `fabricSync` already reaches both sides through `Fabric.Commit`'s detached push and needs no change.
