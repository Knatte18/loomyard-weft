# Card 3 — battenshed-prime-lock-wait

**Edit:**
- `internal/battenshed#PrimeLock`
- `internal/battenshed#worktreeCreateProducer.Call`
- `internal/battenshed#worktreeTeardownProducer.Call`
- `internal/battenshed/create_test.go`
- `internal/battenshed/teardown_test.go`
- `internal/battenshed/doc.go`

**Create:**
- `internal/battenshed/primelockwait.go`

**Uses:**
- `internal/battenshed#waitOrCancel`
- `internal/battenshed#cancelErr`

**Intent:**
`Worktree-Create` and `Worktree-Teardown` wait for a contended prime lock, bounded and cancellable, instead of returning `Stuck` on the first contended try, so parallel batten runs whose create or teardown rows overlap take turns rather than halting `blocked` (Decisions 7, 8).

`primelockwait.go` declares the two package constants, `primeLockPollInterval` (2 s) and `primeLockWaitBound` (10 min), and one shared unexported helper both producers call in place of their single `Acquire`.
The helper calls `PrimeLock.Acquire` and, on contention, sleeps `primeLockPollInterval` through the seam and retries, until the lock is acquired, the context is cancelled, or the bound is spent.
The bound is attempt-counted (`primeLockWaitBound / primeLockPollInterval` sleeps), never read off a wall clock, so a test with a no-op sleep proves it deterministically, the same way `InnerRun`'s attempt cap is proved.
It logs one info line when it first finds the lock contended, naming the producer, the lock path and the slug, and never again for the same call.
An `Acquire` error at any attempt is returned as the hard error it is today, and a context cancelled during the wait returns the producer's existing `cancelErr` diagnosis.

`PrimeLock` gains a `Sleep func(ctx context.Context, d time.Duration)` field, the injected sleep seam; a nil `Sleep` resolves to `waitOrCancel`.
`Acquire` stays non-blocking and its contract text is unchanged; `battencli`'s closure needs no edit, since a nil `Sleep` is the production value.
Both producers keep their existing dispositions otherwise: the release still runs deferred on every exit after acquisition, and a bound exhausted is `Stuck` — never a transient-marked error — with the existing contention reason extended to name the wait, e.g. `prime lock "<path>" is still held after waiting 10m0s; another batten producer is creating or tearing down a task worktree`.
Update both `Call` doc comments, `PrimeLock`'s doc comment (the new field), the `deps.go` header, and `doc.go` with the wait.
`battenshed` stays outside the Fabric Vocabulary Invariant's owner set, so no identifier, message or comment added here names either fabric side.

Tests, over fake seams and a no-op or counting `Sleep` so none spends real time:
- In `create_test.go`, give `fakePrimeLock` a no-op `Sleep` so the existing contention test no longer waits out the real bound, and add: lock contended for N polls then free acquires it, calls `createWorktree`, returns `Done`, and releases; contended past the bound returns `Stuck` with a reason naming the lock path and the wait, `createWorktree` never called; context cancelled mid-wait (the `Sleep` closure cancels it) returns the cancelled-during-run error and not `Stuck`; an `Acquire` error on a later attempt is a returned hard error.
- In `teardown_test.go`, the same three wait cases for `Worktree-Teardown`: contended then free reaches `Shutdown` and `Remove` and returns `Done` with the lock released; contended past the bound is `Stuck` with neither `Shutdown` nor `Remove` called; cancelled mid-wait returns the cancel error.

**ImpactSummary:** Both prime-lock producers in `internal/battenshed` change how they treat contention; `PrimeLock` gains one optional field, so existing constructions in `battencli`, `battenrecipe`, `shedbuild` and `shedrecipe` fixtures compile unchanged.
