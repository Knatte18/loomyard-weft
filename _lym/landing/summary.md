# Worktree-Create narrows its prime lock so creates run side by side

Runs released together queued behind one another in Worktree-Create, because the row held the hub's prime lock across the board claim, the config reconcile and the pair's push.
A slow or refused GitHub push therefore stalled every create behind it.
Worktree-Create now takes the lock only around the steps that need it, so independent creates run side by side.

## What changes

- `battenshed`: `NewWorktreeCreate` no longer acquires the prime lock around the whole create.
  Its create closure now has the signature `func(context.Context, HoldPrimeLock) error` and takes the lock through the hold it is handed.
  The lock wait is unchanged: poll every 2 s for up to 10 min.
  A wait that runs out is Stuck, a failure of the lock itself is a hard error, a cancelled context returns the cancelled-during-run error, and any hold the closure leaves taken is released when the row returns.
  `Env.CreateWorktree` in `shedrecipe` carries the new signature.
- `fabricengine`: `AddOptions.Hold` is taken after the origin probes have started and before the fork point is read, and released when `Add` returns.
  The warp and weft origin probes now start in the background before the pre-flight reads and the hold's wait, and `Add` waits for them under the hold before judging their answers.
  Pre-flight refusals keep precedence over a probe's error, and every return waits for the probes so no git call outlives `Add`.
- `battencli`: the create closure passes the hold into `Add`.
  The removal of a held member's pair in `refuseHeldMember` also takes the hold, since it writes the hub's worktree registry.
  The board claim, the config reconcile and the pair's push run outside the lock.
- `docs/overview.md` and the `battenshed` package doc describe the narrowed hold.

## Design decisions

- The hold lives on the closure rather than on the row, because only the closure knows which steps touch the registry or the prime's tree.
  The row keeps the release-on-every-exit guarantee, and `release` is idempotent.
- `Add` calls `Hold` while it holds the pair's teardown lock.
  A hold whose lock holder waits on this pair's teardown lock would stall both until the teardown wait ran out; this is safe only because a pair is never created and removed at once.
  The `AddOptions.Hold` doc states it.
- The hold still covers the probe time left after the pre-flight reads, and the adopt fetch, the fast-forward fetch of a behind branch and the archived-leftover replace.
  Those run under it by design.

## Known limits

- The board claim's push to Bolt stays synchronous: it is the owner-machine claim another machine reads.
- Two Bolt round trips per create are not merged.
- The weft remote-refs lock queue is not changed; it is tracked as note `push-holds-remote-refs`.
- The Open-Window delay of over a minute is not diagnosed here.
- The pair's push is still synchronous in the create, so a slow push delays its own run, though no longer other runs' creates.

## To check by hand

- No test covers the span of the hold inside `Add`: that it is taken after the probes start and before the fork point is read (`add-hold-span-untested`, `internal/fabricengine/add.go`).
- No test covers the create closure's hold placement: that the claim, the reconcile and the push run outside the lock and the registry writes inside it (`create-closure-hold-untested`, `internal/battencli/wire.go`).
- The Goal's ideas left unmade, listed under Known limits, have no recorded decision in the task's own record other than the parent's note (`goal-ideas-unmade-no-decision`); confirm the limits above are acceptable.
- Start several released tasks together on a prime and confirm their creates overlap.
  This was not run.
