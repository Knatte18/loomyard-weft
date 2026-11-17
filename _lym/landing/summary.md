# gate redundant: cap a test run's memory so a runaway mutant ends alone

`lym gate redundant` froze Neve twice on `internal/websterengine`.
A mutant made a loop keep memory for ever, four such runs went side by side, and nothing bounded their memory.
The user slice reached its `memory.high`, where the kernel throttles the whole slice, so lym stalled before its own timeout could fire.

Every test binary that `internal/testcover` runs alone now runs under a memory cap.
That covers `lym gate redundant`, `lym gate testmap`, the redundancy report step and `cmd/testtiming -redundancy`.

## The cap

`proc.RunMemoryCapped` runs a command with the memory of its process tree capped.

- **Linux with a systemd user manager.**
  The command runs under `systemd-run --user --scope --quiet --collect -p MemoryMax=<cap> -p MemorySwapMax=0 --`.
  `systemd-run` moves itself into the scope and then becomes the command, so the pid and the process group stay the command's own.
  The process-group kill of a timeout or a cancel therefore still reaches the run; `TestRunMemoryCapped_ACancelEndsTheRunAndIsNoCapKill` pins that.
  The kernel kills a run that passes the cap, and the call reports a SIGKILL that no cancel of the command sent as that kill.
- **Linux without one.**
  One probe per process starts a capped scope that prints its own cgroup,
  and passes only when that cgroup's parent hands the memory controller down:
  a user manager without the controller starts the scope and caps nothing.
  Where the probe fails, `prlimit` sets `RLIMIT_DATA` on the started process, and the mutation check runs one test at a time.
- **Windows.**
  `createKillOnCloseJob` takes a memory limit and sets `JOB_OBJECT_LIMIT_JOB_MEMORY`.
  A cancel ends the whole job, which is the tree kill on timeout Windows lacked.

`Toolchain.MemoryLimit` puts every test-binary run through it; the go commands are never capped.
`RunResult.MemoryCapped` sits beside `TimedOut`.
It is set for a run the kernel killed, and for a run that failed with the Go runtime's out-of-memory fatal error in its output.
The second case is how the cap shows under `RLIMIT_DATA` and under a Windows job, where the allocation fails and nothing is killed.

`redundancy_test_memory_mb` in `gate.yaml`, default 2048, sets the cap.
It is a cross-pair key like `redundancy_test_timeout_sec`, and `LoadConfig` refuses a value below 1.

## The mutation check

- A run the cap ends kills its mutant, as a timed-out run does.
  It is never run again alone: each run has its own cap, so it reaches the cap alone too.
  A capped run with no mutant on leaves its test unjudged, as any failed one does.
- Fewer runs go side by side when memory is short.
  `proc.ReadMemoryLimit` reads `memory.high` and `memory.max` up the cgroup ancestry and returns the level with the least memory left.
  The caps of the runs together stay within that, and one run always goes.
- The answer's `mutation` section gains `timed_out` and `memory_capped`:
  each run with a mutant on that ended that way, with the mutant's file, line and operator and the test.
- Each run's mutant is logged before the run starts, with its file, line and operator.
  The log of a machine that froze then names the mutant, which it could not before.
- The mutant timeout floor drops from 10 s to 2 s.

The redundancy report step lists a test the cap ended as not judged, with the cap as its reason.
`lym gate testmap` records one as failed.
`cmd/testtiming -redundancy` had no bound at all on its test runs; it now ends one after two minutes or at 2048 MB.

## Design decisions

- **`RLIMIT_DATA`, not the `RLIMIT_AS` the board entry names.**
  `RLIMIT_AS` counts address space a process only reserves, and the Go runtime reserves far more than it uses.
  Measured on Neve, a Go process holding 57 MB had 1.3 GB of address space, and the running `lym` processes 2.6 GB at 40 MB.
  A 512 MB `RLIMIT_AS` ended a 16 MB Go program at start-up with `failed to reserve page summary memory`.
  So a 2048 MB `RLIMIT_AS` would fail healthy test binaries, and a limit relative to the size at `prlimit` time would race the runtime's start-up.
  `RLIMIT_DATA` counts the memory a process maps to write to, which tracks what it holds:
  under a 64 MB limit the same program ran while it kept 16 MB, and died with `runtime: out of memory` when it went for 200 MB.
- **The side-by-side clamp is in `lym gate redundant` alone.**
  `lym gate testmap` runs the suite's own tests, where several runaways at once are not expected, and the report step runs one test at a time.
  Both still get the cap.
- **The file cache does not count as used memory.**
  A slice's `memory.current` sits near its `memory.high` on a machine that builds, because the cache fills what is free.
  `ReadMemoryLimit` subtracts `inactive_file` and `active_file`, so the clamp sees the memory the kernel cannot drop.
- **A timed-out covering run is still run again alone.**
  A timeout beside other runs can come from load, which a run alone settles; a cap cannot.

## Bounds

- The SIGKILL test also reads a kill from outside lym, such as the machine's own OOM killer, as a cap.
  The run failed either way.
- A hog that is a child of the test is killed by the kernel in its place, and the test then fails or times out without the mark.
  A mutant counts as killed either way.
- `RLIMIT_DATA` bounds each process alone and is set after the command started, so a child the command forks first escapes it.
- A machine with no memory limit in its cgroup ancestry gets no clamp, only the caps.
- The Windows path is cross-compiled and vetted, and its test has not run on Windows.

## Tests

Every fixture that allocates stops by itself at 384 MB, so a cap that does not hold fails its test and never the machine.

- `internal/proc`: a child within the cap ends on its own and one over it does not, on both platforms;
  on Linux a scope reports the kill, the path with no user manager fails the child's allocation, and a cancel still ends a capped run.
  `TestReadMemoryLimitAt` pins the limit reader over fixture cgroup files.
- `TestRunAlone`: a run over the cap returns failed with `MemoryCapped`, and a run under it passes with its arguments and profile unchanged.
- `TestGateRedundant`: the fixture gains `Hoard`, a loop that keeps a megabyte a pass and is endless once its condition is negated.
  The answer lists that mutant under `memory_capped` for both tests that check it, counts it as killed, and its run count holds no second run of a capped one.
- `TestClampParallelToMemory` and the `TestLoadConfig` row for the new key.
