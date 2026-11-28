# gitexec: treat a missing local origin as unreachable

A remote whose repository path is gone from this machine, such as a removed bare repository
or a directory below an unmounted mount point, is now a transport failure, as a refused connection is.
The path may come back, so such an origin should leave work owed or retried rather than stop it as a refusal.

Closes #294

## What changes

`gitexec.IsTransportFailure` now also accepts git's "does not appear to be a git repository"
and "repository ... does not exist" messages when the path they name does not exist.
The change is in that one function, so every caller of it is affected:

- A push to such an origin is retried on the push backoff (unless it is single-attempt),
  then recorded as an owed `transport` failure instead of `other`.
- An origin reconcile retries and then reports `Unreachable` with a nil error instead of returning an error.
  Its callers take their existing unreachable paths.
  For example, `orch start` warns, the loop's pre-push check lets the push go on unchecked,
  `fabric remove` judges from local state, and Finalize reports a transient failure.
- `cleanup --apply` over a missing origin now warns and runs its removal segments,
  where it used to refuse and run none.
- Origin reads during an add are retried on their short backoff.
- A pull whose code-side fetch fails on a missing origin now returns an error that reports a transport failure.
- `shedtransient.Class` classes the error as a git transport failure, so the steps that retry transient errors retry it.

## Design decisions

- The decision rests on the filesystem as well as on stderr: git prints the same message for a missing path
  and for an existing directory that holds no repository.
  Only the missing path counts, so a directory holding no repository stays a refusal.
  A link whose target is gone counts as missing.
- A name with no path separator is never read as a path, because git names an unconfigured remote that way.
- Existing tests that meant "an origin that is no repository" by a nonexistent path now use an existing directory.

Known limits, also stated on `IsTransportFailure`:

- A mount point left behind as an empty directory reads as a refusal.
- A relative remote path with no separator reads as a refusal.
- A relative path is read against the directory git ran in,
  which matches git for push, fetch, pull and ls-remote only at the worktree top.
- An ssh remote naming a server path that is also absent on this machine reads as unreachable.
- A mistyped or deleted local origin path is now treated as owed or retryable, not reported as a refusal.

## To check by hand

- An unconfigured remote name that holds a path separator now reads as unreachable
  when no such path exists (`namesMissingLocalPath` in `internal/gitexec/transport.go`).
  Decide whether that is acceptable.
- `TestIsTransportFailure` in `internal/gitexec/transport_test.go` now creates a symlink and fails if it cannot.
  On Windows that needs the symlink privilege.

## Merge danger

**Door:** two-way
**Blast radius:** fabric
