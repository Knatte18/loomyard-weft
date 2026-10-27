# Dobby survives deploys, logs changes only; adopt check gets its own gate

A dobby now survives a deploy by re-exec and never ends silently.
Its log records only changes plus a ten-minute heartbeat, and stays within a size bound.
A dobby counts and tidies only the landings its own machine made.
The adopt machine check runs under its own gate key `adopt`, reports to the orch and shows in `lym beam status`.

## What changes

**Dobby start and exits (`internal/dobbyengine`, `internal/beamcli`, `internal/selfexec`).**
`runDobby` no longer pre-checks `dobbyengine.Live`; the dobby takes its lock first, and a re-exec'd dobby waits longer for it, so a momentary probe or the old process still exiting no longer makes the new one give up.
Every way a placed dobby ends, or fails to re-exec, leaves a line in `dobby.log`, and the abnormal ones post to the orch.
A beam dobby whose worktree is gone ends and stops its tidy agent.
A tier1 test replaces the binary under a running dobby.
Closes #880

**Dobby log.**
`dobby.log` holds only the dobby's own lines, written line-atomically, changes only plus a ten-minute heartbeat, and is rotated to one previous file at a size bound.
The detached process's raw stdout and stderr go to `dobby.out`.

**Per-machine tidy (`internal/shedengine`).**
A run's Land history entry records the machine that ran it, and the dobby's landing count reads only this machine's landings, so no landing is tidied on two machines.
Landing lists written before the machine stamp existed are emptied once.

**Adopt check (`internal/adoptcli`, `internal/gatecli`, `internal/gateslot`).**
`gateslot` reads a new key `adopt` (template `{tiers: [0], scope: all}`).
`lym adopt` spawns the hidden `lym adopt check`, which runs the gate in-process, writes `check.json` beside `suite.log`, posts a line per tier and a final green or failed notice to the nearest live orch, and holds a lock so only one check runs at a time.
`lym beam status` shows the check: running with the tier in flight, green or failed, or `ended without a verdict` when the recorded process is gone.

## Design decisions

- The reconcile copies a hub's tuned `finalize` into `adopt` when the hub `gate.yaml` has no `adopt`, so existing hubs keep the tiers they run today.
  It lands through the reconcile only, never as a template fill.
  A hub returns to the template's `adopt` by setting it through `lym config gate`, since deleting the key just gets `finalize`'s value copied back.
- The check runs the gate in-process rather than shelling out, keeping one gate function and parsing no output.
  It calls `hubreconcile.Ensure` first; a failed reconcile ends the check as `failed` before any tier runs, so no check goes green on the template's tiers.
- The lock beside `check.json` serializes checks; gate slots alone do not, since a hub may hold more than one.
  Between a spawn and the child taking the lock, `lym beam status` can show the previous record, and the pid in a second `lym adopt` envelope can name a check that lost the lock.
- Not done: keeping the flock across the exec by fd inheritance, a window or strand for the check, and tidying on one machine the landings another machine never tidied.

## To check by hand

- A kill from outside (SIGKILL, OOM) still leaves no line, only the heartbeat's silence; the original disappearance in #880 could not be ruled out as one.
- A refusal at `filepath.Abs` or `lymcwd.ResolveWorktree` in `runDobby` leaves only its envelope on stdout, no `dobby.log` line.
- A real deploy over a running dobby, and a real `lym adopt` on a hub with a tuned `finalize`, were not run end to end here; check that the dobby re-execs and that `adopt` carries `finalize`'s tiers.
