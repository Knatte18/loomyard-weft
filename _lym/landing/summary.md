# Reap lym's child processes, and read a zombie as gone in the registry

Long-lived lym processes piled up hundreds of `[lym] <defunct>` children, and `lym tender status` listed them as live owned `batten` processes under slugs that had landed long before.

## Cause

On the live hub, every zombie under the prime's dobby was older than that dobby's last re-exec.
Dobby waits on each batten pass in a goroutine, and `lym orch watch` waits on its children the same way.
When either process re-execs onto a new lym binary, the kernel keeps its children but the goroutines that waited on them are gone, so the new image never reaps them.
A second, smaller source: several detached starts never waited at all.
These were the board sync, the detached push, the front-task action, the dobby spawn, the reed watchdog and remove, the adopt check and the terminal window.
The front-task start called `Process.Release`, which does not reap a child on Linux.
Inside a long-lived starter each of these left a zombie per spawn.

The registry kept them visible: a zombie still answers signal 0 and its stat keeps the registered start time, so `runproc.List` read it as live.

## Change

- `proc.Reap` waits on a started command in the background.
  Every detached start that never waited now calls it, and the front-task start uses it in place of `Process.Release`.
- `proc.ReapInheritedChildren` waits in the background on every child the calling process already has.
  `lym beam dobby` and `lym orch watch` call it first thing.
  A fresh start has no children, so it does nothing; after a re-exec it adopts what the old image left, and logs the pids it reaps.
- `runproc`'s listing requires a running pid, `proc.IsRunning`, so a zombie record is pruned like a dead one.
  `lym tender status` and `lym tender reap` read the registry through it.
  The end sequence's lister uses the same seam.
- A new `PATTERN-reap-children` entry states the rule: a starter waits on its child, by `cmd.Wait` or `proc.Reap`, and a process that re-execs itself calls `proc.ReapInheritedChildren` before it starts a child.
  The `PATTERN-spawn-observability` background, and the `proc`, `runproc`, `selfexec`, `beamcli` and `orchengine` package docs, now describe the reaping.

## Design decisions

- `ReapInheritedChildren` takes the pid set at the moment of the call and waits on those pids alone, so it must run before the process starts its own child: called later, it would race that child's own `Wait`.
  Both callers run it in the cobra `RunE`, ahead of all wiring, and not in `runDobbyWith`, which tests drive in a parallel test binary.
- A child that is mid-run when its starter re-execs keeps running.
  The new image only adopts its wait, and a batten pass's exit is not delivered to the new dobby, which reads run state from disk as before.
- The detached starts are reaped rather than double-forked, so the existing `runproc` registration, Setsid detachment and pids stay as they were.
  In a short-lived CLI the starter usually exits first, and init reaps the child.

## Tests

- `TestReap` starts `true`, reaps it, and waits until its pid no longer answers a signal.
- `TestReapInheritedChildren` starts an exited child and a running one without waiting.
  It checks that `ReapInheritedChildren` returns both, and that each disappears once it exits.
  The test is not parallel because it waits on every child of the test process.
- `TestRegisterAndList` gains a zombie case: a record whose start time matches and whose pid is not running is pruned.
  The end-sequence fake now reports a zombie entry as not running.
