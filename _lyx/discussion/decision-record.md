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

- Batten's status seam: it runs in the prime, whose code branch is the parent branch the operator pushes; it keeps pushing the records side only, though it gains the lock and the retry through the shared primitive.
- Any change to agent prompts, including Burler's "Never push": agents keep committing with `git commit`, and Go pushes at the next transition.
- Auto-resolving a real divergence (rebase, merge or force push) on either side.
- Publish and Finalize pushes, which already push the task branch.

## Decisions

1. **One both-sides anchored push in `fabricengine`.**
   A new vocabulary-neutral entry beside `PushAnchored` (name chosen at Plan, e.g. `PushPairAnchored`) takes a `*lyxcwd.Location`, resolves both worktrees itself and pushes the code side, then the records side, each through `gitrepo.PushRebaseFree`, under the absorbing push lock `CoalescePushBothAt` takes (`fabric.push.lock` under the records worktree's `.weft/`).
   It returns `gitrepo.ErrPushRejected` unwrapped when either side stays rejected after the retry of Decision 4, and every other push error as an error; it attempts the records-side push even when the code-side push failed, and reports both failures.
   It records each advanced side in its `MutationRecord`.
   Rationale: loomcli may not name the sides (`PATTERN-fabric-vocabulary`), all git stays in `fabricengine` (`PATTERN-fabric-git`), and rebase-free keeps agents' SHAs and the correspondence index intact.
   Records-side push regardless of the code-side outcome keeps a rejected or detached code branch from also stranding the run's status.
2. **Loom's status seam and `commit-records` use it.**
   `loomCommitStatusDeps`'s `Push` calls the new entry instead of `PushAnchored`.
   The statuscommit core's push-warns disposition is unchanged: every push error, the rejection included, warns and the run continues; `commit-records` keeps its error envelope on a push error.
   Rationale: the transition seam is the one place Go pushes after every step, so the code branch lags the remote by at most one step.
3. **`PushAnchored` takes the absorbing push lock too.**
   Batten's seam keeps calling it, records side only.
   Its doc comment's "taking no lock is deliberate" reasoning is replaced: the lock is the weft-side `fabric.push.lock`, not `PushCoalesced`'s repo-root lock, and contending with the detached push child is the point.
   Bound: a status push now waits for an in-flight push in the same pair or prime, at most one push's duration per holder; the lock is blocking with no timeout today, as `CoalescePushBothAt`'s already is.
   Rationale: the most likely cause of #418 is the lock-free status push racing a detached `lyx fabric push` child on the same records branch, where git reports the losing ref update as rejected.
4. **One retry on a rejection that is not a divergence.**
   On `ErrPushRejected` for a side, the primitive fetches that side's upstream; when the fetched remote tip is an ancestor of, or equal to, local HEAD, it pushes once more; otherwise it returns the rejection.
   Bound: one fetch and one extra push per side per call, never a rebase, merge or force; a remote holding commits the local branch lacks is still left alone and warned about, since another machine advancing the branch is the operator's decision.
   The retry lives in the new entry and in `PushAnchored`, not in `gitrepo.PushRebaseFree`, whose callers keep its plain contract.
   Rationale: it absorbs a race with any pusher outside the lock (an agent's own `git push`, a transient remote ref-lock failure) without touching history.
5. **`lyx fabric push` and `lyx fabric sync` push both sides.**
   `push`'s normal mode pushes through the both-sides lock path instead of `Fabric.PushWeft`; `sync` passes the code-side path to the detached child as `Fabric.Commit` already does.
   Bound: in the prime this pushes the parent branch's unpushed commits, which `lyx fabric commit` already does whenever it lands something; nothing new in kind.
   Rationale: the three verbs agree, and `lyx fabric push` is the way forward `commit-records` names after a failed push, so it must carry the task branch.
6. **New PATTERN entry under "Fabric and git".**
   `PATTERN-push-both-sides` — Pushing a task pair's records side from Go: the code branch goes with it, rebase-free, under fabric's absorbing push lock; pushing is never left to an agent.
   Enforced by review; it lands in the same commit as the code.
   Rationale: the operator's direction that the remote must not rest on agents is cross-cutting and outlives this task.

## Constraints

- `PATTERN-fabric-git`, `PATTERN-gitrepo-client-boundary`, `PATTERN-gitexec-checked-call`: all git through `fabricengine`/`gitrepo`; the fetch for the retry goes through an existing or new `gitrepo` remote call, never raw git elsewhere.
- `PATTERN-fabric-vocabulary`: loomcli, battencli and statuscommit name no side; the new export's name carries neither side.
- `PATTERN-mutation-record`: the new push result embeds `MutationRecord`, appending a push only after a side observably advanced.
- `PATTERN-no-denied-recovery`: no refusal or warning names `git push --force` or `git reset --hard`.
- `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-hubforge-fixtures`: push tests that need a real remote are `integration`-tagged on hubforge fixtures; the seam's dispositions stay covered by stub closures.
- `PATTERN-test-economy`: extend the existing `PushAnchored`, `CoalescePushBothAt`, status-seam and `commit-records` tests before adding files.
- `WEFT_SKIP_GIT` / `WEFT_SKIP_PUSH` (`SyncOptions.SkipGit`/`SkipPush`) skip both sides with no lock taken, as today.
- Docs land in the same commit: `fabricengine`'s doc comments for the anchored pushes, `statuscommit`'s and `loomcli`'s seam comments, `fabriccli`'s `push`/`sync` long help, and the PATTERN entry.

## Auto-mode assumptions

- #418's cause is the lock-free status push racing a detached push child; the drive report was not inspected, so the retry of Decision 4 also covers other non-divergent rejections.
- "Wherever the weft is pushed after a step" means loom's transition seam and `commit-records`; mid-step agent commits may stay local until the step's next transition.
- Batten in the prime keeps the parent branch out of its automatic pushes.

## Open risks

- An agent that amends or rebases an already-pushed commit makes the code side diverge from its remote; the push then warns every transition and Publish's push is rejected until the run's own `lyx webster reset` or the operator resolves it.
- The blocking lock has no timeout: a hung push child (stalled network) holds back every status transition in that pair until git gives up.
- A code-side push from a detached HEAD (mid-rebase) fails and warns each transition until the branch is back.

## Acceptance criteria

- A loom transition after an agent's raw `git commit` on the task branch leaves the remote task branch at local HEAD, along with the records branch.
- `lyx loom commit-records` pushes both sides; a rejected code-side push yields its error envelope, and the records side is still pushed.
- `lyx fabric push` and `lyx fabric sync` advance the remote task branch as well as the records branch.
- A rejection whose remote tip is an ancestor of local HEAD is retried once and succeeds, with no warning logged.
- A real divergence (the remote holds a commit the local branch lacks) is not retried past the one fetch, warns at the status seam, and leaves both local branches' SHAs unchanged.
- A status push and a detached `lyx fabric push` child on the same pair serialize on the absorbing lock.
- `WEFT_SKIP_PUSH=1` pushes neither side and takes no lock.
- Batten's status seam still pushes only the records side.
- The status seam's dispositions (mid-merge skip, commit hard-error, push warns) are unchanged.

## Notes for the plan writer

- Today's pushes: `internal/statuscommit` (shared seam core), `internal/loomcli/wiring.go` (`loomCommitStatusDeps`), `internal/loomcli/commitrecords.go`, `internal/battencli/commitstatus.go`, `internal/fabricengine/pushanchored.go`, `coalesce.go` (`CoalescePushBothAt`, `pushRebaseFreeLogged`), `spawn.go`, and `internal/fabriccli/weft_verbs.go` (`push`, `sync`).
- `CoalescePushBothAt`'s `pushRebaseFreeLogged` swallows a rejection as a warning; the new entry must surface it so `commit-records` can report it.
- `webstercli`'s `fabricSync` already reaches both sides through `Fabric.Commit`'s detached push and needs no change.
