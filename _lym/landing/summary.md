# Serialise remote-tracking ref writes under a per-repository lock

A batten pass could fail its origin reconcile with `incorrect old value provided` when another lym process fetched or pushed the same remote-tracking ref at the same moment.
It could also judge a branch rewound against origin because it read origin while the writer's own push was still in flight.
Either way the driver spawned a tackler that had nothing to fix, or that could not tell a real rewind from a push in flight.

Every lym git call that writes a repository's remote-tracking refs now runs under one lock and waits for it while it is held, instead of failing or retrying.

## What changes

- `internal/gitrepo/remoterefslock.go` adds the remote-refs lock: `lym-remote-refs.lock` (`RemoteRefsLockFileName`) in the repository's common git dir, so it orders every worktree of the repository.
  `Repo.HoldRemoteRefs` takes it for callers that need to hold it across several steps, and `Repo.FetchWithArgs` is the fetch for callers that name their own remote and refspecs.
- `Fetch`, `Pull`, the plain push paths, the rebase inside the push retry, and the branch-delete and leased-update pushes in `internal/gitrepo` each run git under the lock.
  A push's retry backoff sleeps outside the lock.
- The fetches in `internal/fabricengine` (`add.go`, `weftbranch.go`, `remoteonly.go`, `archive.go`) go through `gitrepo` instead of raw `gitexec`, and so take the lock.
- `ReconcileOrigin` now holds the lock of the code repository and then of the records repository from the `ls-remote` pair through the last branch verdict (`judgeAgainstOrigin`).
  A push in flight lands before origin is read, and a push or fetch that arrives later waits for the verdicts.
- `PATTERN.md` gains `PATTERN-remote-refs-lock`, pinned by a test in `cmd/lym`, so a new fetch, pull or push written outside `gitrepo` is caught.
  A tag push, a `--refmap=` fetch and a probe-clone fetch write no tracking ref and are exempt.

## Design decisions

- **A repository-wide lock, not the existing push lock.**
  The push locks are per checkout, so they do not order two worktrees of one repository, and `PushCoalesced` holds its lock across a loop that fetches, so reusing it would deadlock in-process.
  The per-checkout push locks are unchanged.
- **The lock is not reentrant.**
  A holder of `HoldRemoteRefs` runs no `gitrepo` fetch, pull or push, since each takes the lock itself.
  The reconcile's own `fetchMovedHeads` runs bare git for that reason.
- **An unreachable origin blocks other fetches and pushes for the retry bound.**
  The reconcile keeps both locks through the `ls-remote` retries and their sleeps, so an unreachable origin holds up every fetch and push of both repositories for at most `RetryBound`.
- **`resolveArchiveTip` fetches by SHA.**
  It now picks the exact `refs/heads/<branch>` line from `ls-remote` and fetches that SHA, because `FETCH_HEAD` can be rewritten by another fetch once the lock is released.

## To check by hand

- The lock also covers pushes, `pull --rebase` and branch deletes, which goes beyond the original wording that only fetches and the reconcile's `ls-remote` take it.
  Confirm that is the intended reach (`internal/gitrepo/remoterefslock.go:11`).
- A real two-process race between a landing and a batten pass was not reproduced; the tests pin lock waiting and release, not the original timing.
