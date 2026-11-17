# gate redundant: a runaway mutant eats all memory and freezes the machine

`redundant-mutant-memory-cap` · Note · bug, gate

## Seen (2026-11-16 and 2026-11-17, Neve)

`lym gate redundant`, since its mutation check landed in `e97e2474`, froze Neve twice; both times the power key was the only way out.

- The task `redundancy-confirmed-survey` ran it once per package over the hub.
  On 2026-11-16 the machine froze at 15:47 while the loop was on `internal/websterengine`.
- After the reboot on 2026-11-17 the loop resumed at `internal/websterengine`, and the machine froze within a minute.
  The memory sampler's last line, at 07:04:30, shows `/tmp/lym-1000/royti/redundant-*/mutants.test -test.run=^TestRecordBatch_AuditOutcomes$` at 13.3 GB RSS, 9 seconds after it started.
  The user slice has a 16 GB memory limit, and the desktop stopped responding.

A mutant of the code under `TestRecordBatch_AuditOutcomes` makes the test allocate without bound, most likely a loop whose exit the mutation removed.
The mutant's test binary runs with no memory cap and no short time limit, so one runaway mutant takes the whole machine down.

## Idea

Run every mutant's test binary under a hard memory cap and a short timeout, and count a mutant that hits either as killed:

- a memory cap the kernel enforces, such as `prlimit --as` or a cgroup `memory.max` on Linux, with `GOMEMLIMIT` set as well;
- a timeout scaled from the unmutated test's own run time, not the package's `-test.timeout`;
- a cap on how many mutant binaries run at once.

The report names each mutant cut short this way.
Until the fix lands, nobody runs `lym gate redundant` on a package with mutants.
