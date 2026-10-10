# Harden git reads, remote calls and hub gc; refuse done slug in --window

Five git defects stopped or stalled runs; each is fixed at its source.

Closes #468
Closes #458
Closes #447

## What changes

- **Stale pack index (#468).**
  Every go-git use in `internal/gitrepo` now runs inside one helper, `readGoGit`.
  It reruns the whole read once when the error chain holds `plumbing.ErrObjectNotFound` and the pack fingerprint differs from the one the read started under, reindexing first when no other caller already has.
  `Status()`, history walks and tree diffs were bypassing the old per-lookup `lookupObjectRetrying`, which is replaced.
  A gitrepo test pins the helper as the only production caller of `goGit()`.
- **Hung remote calls (#458).**
  `internal/gitexec` runs `push`, `fetch`, `pull`, `clone` and `ls-remote` with `GIT_HTTP_LOW_SPEED_LIMIT=1000` and `GIT_HTTP_LOW_SPEED_TIME=60`, under a 10-minute deadline that kills git's process group on Unix.
  A timeout is a `*GitError` that `IsTransportFailure` reports true for, so existing best-effort and transient paths take over.
  `operation too slow` and `pull` join what `IsTransportFailure` classifies.
  The process-group setup moved from `internal/verifyrun` to `internal/proc` so both share it.
- **Hub auto-gc.**
  `internal/fabricengine` sets `gc.auto=0` and `maintenance.auto=false` on the code and records stores at pair creation and before each housekeeping run.
  After a successful pair removal it runs a best-effort foreground `git gc --auto` on both stores, logged through `internal/logger`; a failure never fails the removal.
- **`batten run --window` on a done slug (#447).**
  `runInWindow` runs the same status check as `battenPreRun` and refuses a done slug with the existing refusal before opening a window.
  The refusal adds "give the new task a different slug" as a way forward.
- **`HeadContains`.**
  The walk is committer-time ordered and stops once a commit is more than 24 hours older than the target's, instead of reading all history when the target is absent.

`PATTERN.md` gains `PATTERN-gogit-read-helper` and `PATTERN-hub-store-gc`; the `doc.go` files of `gitrepo`, `gitexec` and `fabricengine` describe the new behavior.

## Design decisions

- Lock discipline for the helper: the read runs under the shared lock; the exclusive lock covers only the fingerprint check, `Reindex()` and the fingerprint update.
  A reindex by a concurrent caller in the gap still triggers this caller's rerun.
  Composite methods call the helper once per read, so each read gets its own single retry.
- Housekeeping lives in its own `fabricengine` file, outside `destroy.go`.
  It deletes only unreachable objects and expired reflog entries past git's default expiry, and runs with `gc.worktreePruneExpire=never`, so it never touches a worktree's admin entry.
- Timeouts and thresholds are injectable for tests, not operator-configurable.
- `tools/deploy`'s own `push` and `fetch` are left alone: they run only under `update-plugins.sh` in the operator's terminal.

## Known limits

- A done slug stays unusable until board entry `lyx-cleanup` removes finished statuses; `--window` now refuses it by name rather than reusing it.
- `HeadContains` answers false for a commit in HEAD whose committer time is more than 24 hours older than the target's (clock skew); `fabricengine.StencilSource` then logs "behind the build" instead of "source ahead".
- A legitimate remote command over 10 minutes, or under 1000 B/s for 60 s, now fails as a transport failure.
  A step attempt whose best-effort remote calls each time out waits one deadline per call, and the transient re-step doubles that.
- On Windows the kill reaches git only; a `git-remote-https` child may outlive it until its connection fails.
- Hub stores no longer auto-gc for anyone, manual git included; a manual `git gc` still works.
- Unreferenced loose objects survive until git's two-week expiry, so `gc --auto` may repack at nearly every pair removal until then.

## To check by hand

- `HeadContains` (`internal/gitrepo/blobread.go`): a `readGoGit` failure while resolving the target other than object-not-found is wrapped as an error; confirm an object-not-found raised during the history walk is not reported as "absent" and so swallows a walk failure.
- `TestStoreHousekeeping` (`internal/fabricengine/housekeeping_integration_test.go`): confirm the "Remove writes both keys" subtest fails when `Remove` does not write the keys, and does not pass only because `Add` already wrote them.
- Not run: the integration suite results were not available to this description; run `go test ./...` and `go test -tags integration ./...` before pushing `main`.
- Not run: no real dead-connection or SSH hang was reproduced; the deadline is exercised through an injected value only.
