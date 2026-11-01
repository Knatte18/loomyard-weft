# Fetches and pushes of a repository wait under one remote-refs lock

Two lym processes that fetch or push the same branch of one repository both write its remote-tracking ref, and the loser failed.
A batten pass's origin reconcile failed this way with `incorrect old value provided` and spawned a tackler for nothing.
In a second case the reconcile read origin while a writer's push was still in flight and judged a healthy branch rewound.

## What changes

- `gitrepo` gains a remote-refs lock, `lym-remote-refs.lock` in the repository's common git dir, so every worktree of a repository shares it.
  `Repo.HoldRemoteRefs` takes it, waiting while another process holds it.
- `Fetch`, `Pull`, the new `FetchWithArgs` and every push method (`Push`, `PushRebaseFree`, `PushCoalesced`, the remote deletions and the leased update) run each git call under the lock.
  A push's retry backoff sleeps outside it.
- Every fabricengine fetch from origin that writes a tracking ref now goes through `FetchWithArgs` instead of a bare `gitexec.Run`.
  This covers pair creation, checkout, archive, removal, task-branch discard, remote-only commits and remote cleanup.
- `ReconcileOrigin` judges in a new `judgeAgainstOrigin`, which holds the code repository's lock and then the records repository's lock from the `ls-remote` pair through the last branch verdict.
  A push or fetch of either repository waits for the verdicts, and a push already in flight lands before origin is read.
  The reconcile's moves, teardowns and owner reads run after the locks are released.
- `PATTERN.md` gains `PATTERN-remote-refs-lock`.
  The `gitrepo` and `fabricengine` package docs describe the lock, and the gitrepo boundary guard pins `FetchWithArgs`.

## Design decisions

- The record asked fetches to take "the push lock that already orders that repository's pushes".
  No such lock exists per repository.
  `fabric.push.lock` lives under each records worktree, and `.gitrepo-push.lock` lives in one checkout, so neither orders two worktrees of one repository, and both are held across a whole coalescing loop.
  That loop fetches inside it (`retryWeftPush`, `retryBoltPush`), so taking the same lock for each fetch would deadlock in-process.
  The new lock is repository-wide and is held only for each git call, or for the reconcile's read-to-verdict window.
  It sits beside the existing push locks and does not replace them.
- A fetch with `--refmap=` writes no tracking ref and stays unlocked: the remote-leftover probes and the judge-only reconcile's fetch by SHA.
  The records probe fetches into a throwaway repository and stays unlocked too.
  The reconcile's own `fetchMovedHeads` runs while its caller already holds the lock.
- The lock waits rather than failing.
  Only the reconcile ever holds two remote-refs locks at once, always in the same order, so a wait cannot deadlock.

## Tests

- `TestHoldRemoteRefs_OneLockServesEveryWorktreeOfARepository`: a lock held from a linked worktree is held for the primary worktree.
- `TestRemoteRefsLock_ConcurrentFetchesAndPushesOfOneRepositoryAllSucceed`: concurrent fetches and pushes from worktrees of one clone all succeed, and the tracking ref ends at origin's head.
  This reproduces the incident's race, which without the lock fails only some of the time.
- A new `TestReconcileOrigin_Judge` row checks both locks inside every `ls-remote` and fetch the reconcile makes, through the origin call hook, and checks that both are free once it returns.
