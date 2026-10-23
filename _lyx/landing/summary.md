# Orch: count the handoff timeout from the orch's read of the note request

The orch watcher's handoff timeout now starts when the orch reads the note request, not when the watcher posts it.
A soft cycle request that creel holds behind a long orch turn used to spend its handoff time waiting for delivery.
The orch then wrote the note after the timeout had fired, the watcher had returned to idle with `last_abort_reason: "handoff timed out"`, and no cycle ran.
Now the request gets the full handoff timeout from its delivery.

Closes #845

## Changes

- `internal/orchengine`: the `Creel` seam gains `MailReadAt(id)`, the time of the mail's first creel `read` entry, zero while unread.
- `tickHandoff` asks it on each handoff tick until creel answers a time, and records it in the new `State.RequestReadAt`.
  A restarted watcher keeps counting from the recorded time, and a hub pause after the read is still discounted.
- `RequestReadAt` is cleared wherever `State.RequestID` is: `toIdle`, `enter`, the cycle start in `tickIdle` and `ResetForFreshLaunch`.
- `internal/orchcli`: `creelSeam.MailReadAt` folds creel's journal for the mail id into the first `read` entry's time.
  The `refresh` and `distill` help text and the package doc describe the two timeouts a request meets.
- The package doc of `internal/orchengine` describes the handoff-requested phase to match.

## Design decisions

- The earlier clock, the earlier of the phase entry and the request time, is dropped for every trigger.
  A requested cycle's phase no longer times out from the request time.
- A request creel has not journaled as read never times the phase out.
  This is the same wait the clearing phase already has for creel to type `/clear`.
  The same holds while the phase has no confirmed request id yet.
  The cost: an orch stuck in one turn for hours keeps the watcher in handoff-requested as long, where the old rule returned it to idle after the handoff timeout.
  `lyx orch status` shows the phase meanwhile, and a removed strand still ends the watcher through its strand check.
- A failed `MailReadAt` goes through `creelFailed` like every other creel call, so the phase is held and not timed out.
- `State.RequestReadAt` is an added JSON key, read as zero from an older state file.
- The idle phase's stale-marker removal still ages a pending request from its request time; every other phase's timeout is unchanged.

## To check by hand

- `internal/orchengine/watcher.go:1142`: a failed `MailReadAt` lookup holds the phase with `Stuck` set and `StuckByCreel` true, then counts from the read once creel answers.
- `internal/orchengine/watcher.go:1097`: a hub pause opened after the read delays the timeout by the paused time.
- `internal/orchengine/state_test.go:310`: `RequestReadAt` is zero after `ResetForFreshLaunch`, `enter` and the `tickIdle` cycle start, not only after `toIdle`.
- `internal/orchengine/doc.go:190`: the doc says the phase does not time out while its note request has no creel read, a request not yet posted included.
- Run a soft cycle request against a live orch in a long turn and confirm the cycle runs after delivery; this was not run.
