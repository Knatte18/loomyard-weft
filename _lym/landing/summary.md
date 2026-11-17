# beam: `lym beam stop --all` takes the hub down on this machine

Today the operator stops a machine with `lym beam stop` per worktree, `lym reed down` per pair and `lym tender reap`.
`lym beam stop --all` replaces that with one verb, and reed's `prime` button group gains a `stop` entry that runs it behind a confirm text.
The stop writes no tracked record and no per-run pause or cancel intent, so no other machine reads anything from it.

## What changes

- `lym beam stop --all` re-starts itself detached in the prime, through `selfexec` with the spawn logged,
  so the end of the session it was run from does not cut it.
  The caller prints the detached process's pid; the detached process prints a report of each step.
  A slug beside `--all` is refused with a way forward.
- The detached process runs this sequence:
  1. Pauses the hub and waits for hibernate's confirmation that every session is idle, no network call is in flight and no land lock is held.
     A confirmation not reached within hibernate's wait does not stop the stop; the report names what was still busy.
  2. Takes down each task pair's processes and reed session through `lym reed down`'s path, which accepts a pair a hub pause covers.
  3. Runs `orchcli.BeamDown` for the prime and each local beam.
  4. Ends the beams' reed sessions.
  5. Ends run processes no run owns, as `tenderengine.Reap` does.
  6. Lifts its own pause entry through `tenderengine.Lift`, if it added one.
  7. Ends the prime's reed session last.
- It ends every worktree of the hub on this machine, whoever owns it.
  A standing Bolt hold stays, and an operator's or hibernate's hub pause that covered the stop stays too.
  A failed step is recorded in the report and the sequence goes on; the verb then exits with an error naming each failure.
- `orchcli.BeamDown` now ends the orch's registered Claude process through `runproc.EndSubject` as its last step,
  so `lym beam stop <slug>`, a beam's cancel and pawl's Beam-Down row end it too.
- `hibernateengine`'s pause-and-confirm is one exported function that hibernate's carrier and the stop both call.
  It owns only an entry it added.
- Reed's `prime` button group in both templates gains a `stop` entry running `lym beam stop --all`, with a `confirm` text.
  A hub that sets `buttons.prime` in its own layer replaces the group whole and adds the entry by hand.
- `lym reed down`'s pair end in `internal/reedcli` is exported for the stop to call.
- Docs: `docs/overview.md` and the doc comments of the touched modules.

## Design decisions

- The stop builds on the run intents' pause rather than a per-run intent.
  A pushed per-run intent would read as `paused` on other machines.
  Every run is left at a transition pushed or owed: the cut agent step runs again, and an owed push is pushed by the catch-up at the next dobby start.
- The stop lifts its own pause entry because hub-wide `lym tender resume` restarts nothing itself.
  The lift pays the detached pushes the pause deferred, and those run to their end after the stop.
- A stop that dies midway leaves its pause entry; `lym tender resume` lifts it.
- After a stop, `lym beam start` or `lym s` brings the hub back.
  A run whose loop was ended halts `loop-exited` at its first pass, and the orch settles it as after a crash.
- The stop touches no `internal/dobbyengine`, `internal/machinecli` or `internal/termwindow` code.

## To check by hand

- Run `lym beam stop --all` on a real hub, from the prime's stop button and from a beam's session.
  Check that no orch strand, orch Claude process, dobby, coverage upkeep, run process or reed session of the hub remains, and no pause entry of the stop's own.
- Run it under a standing Bolt hold and under an operator's hub pause, and check that each is left as it was.
- Run `lym beam stop --all` then `lym beam start --all`, with a run cut mid-step.
  Check that it halts `loop-exited` and its pushed transitions stand.
- Check that `lym beam stop <slug>` and a beam's cancel end the worktree's registered orch Claude process.
- Terminal windows are not closed by the stop; the operator closes them by hand.
