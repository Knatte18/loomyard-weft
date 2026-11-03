# hibernate: the prime's dobby carries the request, not its own daemon

`lym hibernate request` no longer spawns a detached daemon to carry out the request.
It records the request and returns; the prime's dobby takes the request up and runs the same sequence as before: trigger wait, hub pause, confirmation, hibernate, wake, reconcile and notice.
Dobby already drives every run of its worktree, so the request no longer needs a daemon of its own polling the runs.

## What changes

- New `hibernateengine.Carrier` (`internal/hibernateengine/carrier.go`).
  On each `Tick` with no sequence of its own in flight, it takes up the pending request under the request lock and runs the sequence in a goroutine of its own, so the dobby's ticks never wait on it.
  It takes up two kinds of request: one no live process carries, and one that names the running process itself, which a re-exec leaves behind.
  It leaves alone no request, a terminal one, and one another live process carries.
- `beamcli` wires the Carrier into the prime's dobby only, through the engine's existing `Deps.Tick` hook, so the request is watched and cancelled in one place.
  No beam's dobby carries it or votes on it.
  The prime's file watch also covers the request file, so a written request wakes the tick that takes it up.
- The request stays on disk and keeps its `daemon_pid` and `daemon_start` keys, which now name the dobby that carries it.
  A dobby that was down or died takes the request up when it next ticks:
  - a waiting request goes on as it was;
  - one cut in pausing or confirming has its owned pause entry lifted and goes back to waiting;
  - one cut in hibernating or resuming ends failed with one notice, its entry lifted only when the request resumes on wake.
- `Submit` writes the request with no carrier, refuses any pending request whether or not a live process carries it, and reports `dobby_live`.
  When the prime's dobby does not run, the envelope names `lym beam start root`.
- `Cancel` no longer kills a process.
  The sequence sees the cancel at its next read of the request and ends.
  It then lifts by identity the entry its pause added, which covers an entry recorded after `Cancel` read the request.
- While a sequence is past its trigger wait, the prime's dobby skips its binary re-exec (`holdReexec` over `Carrier.HoldsReexec`), so a re-exec never cuts a pause, a confirmation, a hibernate or a wake.
  A sequence still in its trigger wait holds nothing, since the re-exec takes it up again at the same wait.
- The wait for every session to pause moves over unchanged, the fork rule included.
- Removed: the hidden `lym hibernate watch` verb, the daemon spawn and kill, and the `watch.log` file.
- `lym fabric anchor --set` for the prime is refused while a request is pending in a running prime's dobby; an ended request no longer blocks it.
- Docs updated: the package docs of `hibernateengine`, `hibernatecli` and `beamcli`, `docs/overview.md`, `pattern/PATTERN-daemon-wakeups.md`, and the `fabriccli`, `cleanupcli` and `lymdirs` wording that named the daemon.

## Design decisions

- The sequence runs in its own goroutine rather than being spread over dobby ticks.
  This keeps the sequence, its timings and its tests unchanged.
  It also keeps a slow confirmation, up to sixty minutes with forks, from stalling the dobby's passes.
- The JSON keys stay `daemon_pid` and `daemon_start`.
  An old-binary daemon still running when the new binary deploys reads as a live carrier, so the prime's dobby waits for it to exit instead of carrying the same request twice, with no compatibility code.
- A hub pause leaves the dobby that carries the request outside it.
  The dobby is no session, so tender's pause interrupts nothing in it.
  The sequence runs apart from the tick steps that a pause holds back.
  The only network calls it makes are the wake reconcile's, which pass the gate under the existing exemption of the request's own entry.
  So the pause the request adds never stops the wait.
- Taking over a request cut mid-pause starts it again from the trigger rather than resuming the confirmation, as `Submit` did before for a dead daemon's request.

## To check by hand

- Not run: a real `lym hibernate request` on a Linux machine with the prime's dobby running.
  Check that the dobby takes the request up within a second of the write, that the hibernate and wake run, and that the notice reaches the orch.
- Not run: `lym beam stop root` while a request waits, then `lym beam start root`.
  Check that the restarted dobby takes the request up and fires it.
- Bound to judge: `Deps.Tick` runs after the dobby tick's other steps.
  While an earlier step fails every tick, a new request is not taken up; a sequence already in flight runs on.
