# The hibernate request is carried by the prime's dobby

`lym hibernate request` no longer spawns a daemon of its own.
It records the hub's one request on disk and returns; the prime's dobby takes the request up and carries it out.
The sequence is the one the daemon ran: wait for the trigger, pause the hub through tender's engine, confirm every session paused, hibernate the machine, detect the wake, reconcile with origin, and post creel mail to the orch.
The hidden `lym hibernate watch` verb, the detached spawn and the kill of the daemon are gone.

The change makes dobby the one place that already reads the runs of every beam and drives them to their end, so the request is watched and cancelled there and `battenrecipe.PastLand` stays the one shared past-Land test.

## What changes

- `internal/hibernateengine` gains `Carrier` (`carrier.go`).
  `Tick` takes up a pending request that no live process carries, or that names the running process after a re-exec, and runs its sequence in a goroutine, so dobby's ticks never wait on it.
  Taking up a request left in the pausing or confirming phase lifts the pause entry it owns and returns it to waiting.
  A request left in the hibernating or resuming phase ends failed, since its wake can no longer be seen; the resuming reason says the wake reconcile may not have run and that `lym cleanup --apply` runs it.
- `Submit` returns a `SubmitReport`: the request and whether the prime's dobby runs.
  When it does not, the report carries the way forward `lym beam start root`; the request stays on disk and is taken up once that dobby starts.
  A pending request is refused whether or not a process carries it.
- `Cancel` no longer kills a process.
  It ends the request and lifts the entry the request records.
  The sequence sees the cancel at its next step and lifts, by identity, any entry its pause added after the cancel read the request.
  A request a live carrier holds in the hibernating phase is still refused.
- `internal/beamcli` wires the carrier into the prime's dobby only, never a beam's.
  It sets the dobby's `Tick` hook, adds the request file to the dobby's file watch, and creates the hibernate directory first so that watch can open it before the first request exists.
- `internal/hibernatecli`, `internal/fabriccli` and `internal/cleanupcli` follow: `status` and `lym fabric anchor` report on the prime's dobby, and the anchor move for the prime is refused while a request is pending.
- `resume_on_wake`, from the hub-wide `hibernate.yaml` or `--resume-on-wake`, is kept on the request and honoured at the wake unchanged.
- The wait for every session to pause moves over unchanged, the fork rule included.
- `docs/overview.md` and the package docs describe the new carrier.

Closes nothing: the board entry names no issue.

## Design decisions

- The prime's dobby carries the request, so a request with `--after-all` sees every beam's runs.
  No beam's dobby carries or votes on it.
- The request file is the only state.
  A dobby that is down or dies mid-wait loses nothing; the next prime dobby to start takes the request up again.
- A re-exec of the dobby is held off while a sequence is in flight past its trigger wait, until the sequence has lifted its entry and sent its notice (`Carrier.HoldsReexec`, applied through `holdReexec` in `internal/beamcli/dobbydeps.go`).
  A sequence still waiting for its trigger holds nothing, since the re-exec takes it up again from the same wait.
- When a sequence ends, it rewrites the request file as it stands so the file watch runs the next tick.
  A request written while a cancelled sequence was still ending would otherwise find no tick to take it up.

## To check by hand

- The re-exec hold ends when the sequence returns, which is after the notice is sent.
  Check that the hold cannot end before the notice in the paths where the notice is retried or written to the notice file; the blocked-notify test in `internal/hibernateengine/carrier_test.go` pins the main path.
- Run a real `lym hibernate request` on a Linux machine with a running prime dobby: the request is taken up, the hub pauses, the machine hibernates and the wake reconciles.
  This was not run; the tests drive the sequence through seams.
- Restart the prime's dobby while a request waits for its trigger, and again after the pause is added, and check the request is taken up as described above.
- Check that a hub pause, which the request adds itself, never stops the dobby that waits on the request.
- Request with the prime's dobby stopped, and check the envelope names `lym beam start root` and that starting it takes the request up.
