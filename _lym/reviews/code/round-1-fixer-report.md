# Fixer-report, code review round 1

## Changes

Every finding is fixed as proposed unless marked.

- F1: `PATTERN.md` entry `PATTERN-remote-refs-lock` narrowed to calls that write remote-tracking refs; a tag push, a `--refmap=` fetch and a probe-clone fetch are named as outside it.
- F2: `internal/gitrepo/remoterefslock_integration_test.go` gains a table test over `Fetch`, `FetchWithArgs`, `Pull`, `Push`, `PushRebaseFree`, `DeleteRemoteBranch`, `DeleteRemoteBranchLeased` and `UpdateRemoteBranchLeased`: each call must stay blocked while `HoldRemoteRefs` is held and complete after release.
- F3: fixed with an own change, marked so. The reviewer offered a test seam or a blocking assertion; the fix adds a verdict hook (`originVerdictHook`, `SetOriginVerdictHookForTest`) called before each verdict in `judgeAgainstOrigin`, and the lock subtest asserts both locks are held at every verdict. The blocking-assertion alternative would not fail on a release right after `fetchMovedHeads`.
- F4: `PushRebaseFree` doc no longer says "Lock-free".
- F5: fixed with an own change, marked so. The reviewer proposed fetching into a caller-owned ref or a new locked gitrepo call. `resolveArchiveTip` instead fetches by SHA the tip `ls-remote` listed (the line naming exactly `refs/heads/<branch>`) and returns it, so no `FETCH_HEAD` is read. The repo already fetches by SHA in the judge-only path (`fetchMovedHeads`), and the fetch's error text is unchanged.
- F6: `judgeAgainstOrigin` doc states the locks are held through the ls-remote retries, bounded by `RetryBound`.
- F7: new guard `cmd/lym/remoterefslock_test.go`, an AST scan of `internal/fabricengine` non-test source that fails on a `[]string` holding `"fetch"` without `"--refmap="`, outside a per-function allowlist (`probeMainWeftHasPath`, `fetchMovedHeads`).
- F8: a subtest asserts both locks are free after a judge-only reconcile, an unreachable origin and an origin that is no repository.
- F9: `PATTERN-remote-refs-lock` is marked `(test)`; no background file, since the entry stays one line.
- Self-check fixes: removed the stale `resolveArchiveTip` bullet from `internal/fabricengine/doc.go`; reworded its "lock-free `PushRebaseFree`" line; the guard's allowlist went from per-file to per-function (the gate repair below keeps one file-keyed entry) and its scan from "first element" to "any element" is `fetch`.

## Gate repair

The verify run failed `TestEnforcement_FabricVocabulary` in `internal/lymcwd`: `cmd/lym/remoterefslock_test.go` named a function with a bare weft token in its allowlist key.
The `recordsprobe.go` entry is now keyed by file, whose only fetch is the probe clone's; the `fetchMovedHeads` entry stays per function.
`cmd/lym` and `internal/lymcwd` pass at tier 0.

## Disputed

Disputed findings are fork findings only.

- every fix worse, fix-proposal LOW on F2 (the 500 ms window is probabilistic): `lock.TryAcquireWriteLock` and flock expose no waiter signal, so a deterministic not-returned check needs a production seam inside `lock`; the window can never fail a correct implementation, only miss a missing lock on a very slow machine. Rejected for that reason, evidence `internal/lock` has no waiter hook.
- every fix worse, fix-code-correctness LOW (a branch that moves between `ls-remote` and a by-SHA fetch fails on protocol v0/v1 servers): the alternatives either re-open the `FETCH_HEAD` race or write a shared tracking ref; the repo already relies on by-SHA fetch in `fetchMovedHeads` (`originreconcile.go`, judge-only branch). The failure is a returned error with the same text, never a wrong tag.
- premise false, fix-proposal LOW on the entry versus `fetchMovedHeads`: the entry's last clause, "a judge of origin against tracking refs holds it from reading origin to its verdict", covers `fetchMovedHeads`, which only runs inside `judgeAgainstOrigin` under that hold.

## Deferred

Nothing deferred.

## Self-check

- fix-consistency, MEDIUM, stale `doc.go` bullet for `resolveArchiveTip`: fixed as proposed.
- fix-affected-set, LOW, same stale bullet: duplicate, fixed.
- fix-affected-set, LOW, "lock-free, rebase-free `PushRebaseFree`" in `doc.go`: fixed; the test message at `pushrebasefree_integration_test.go:57` and `pushanchored.go:69` are left, since both speak of the single-pusher lock or a generic push and stay true.
- fix-proposal, LOW, F6 wording: fixed, and fix-code-correctness LOW, same wording: duplicate, fixed.
- fix-proposal, LOW, F1/F7 allowlist file-granular and first-element scan: fixed (per function, any element); the claim that the entry disagrees with `fetchMovedHeads`: disputed, premise false.
- fix-proposal, LOW, F2 window: disputed, every fix worse.
- fix-code-correctness, LOW, by-SHA fetch after the branch moves: disputed, every fix worse; the tail-match part: fixed with an exact ref match.
- All four forks returned a report.
