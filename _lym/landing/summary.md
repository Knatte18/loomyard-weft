# Memory guard for every test binary lym runs, and a per-tier memory cap

The gate watchdog sampled only CPU, and only for `lym gate test`'s own `go test`.
A test that allocates without bound in a normal gate run could freeze the machine the way a runaway mutant did.
This change adds a memory guard over every test binary lym starts, and lets a tier run its command under the kernel cap that `redundant-mutant-memory-cap` built.

## What changes

- `internal/gatecli`: a memory guard (`memoryguard.go`) samples the process tree once a second.
  - It kills a `.test` process's tree, the binary and every process it started, once their resident memory together passes `memory_quit_mb`, with SIGKILL.
    A tree that holds another tree over the cap is spared, and the inner binary is killed.
  - With no tree over the cap, it kills the largest tree of 1024 MB or more once the memory limit lym sits under has less than a tenth of its Max left.
    `proc.ReadMemoryLimit` reads that limit; on Neve it is the user slice's `memory.high`.
  - Each kill prints a line naming the binary and pid, its package directory, read from the binary's working directory, the `-test.run` value it was given, and the reason.
  - In `lym gate test` the guard runs over each tier command beside the CPU watchdog and the process cap.
    The kill lines go into the tier's log and its failure report's causes, the run fails, and it is never rerun.
  - `lym gate redundant` and `lym gate testmap` run the same guard over their own process tree for the whole call, and print each kill line to stderr.
    They still cap each test run at `redundancy_test_memory_mb` through the kernel.
  - The guard is off when `memory_quit_mb` is 0, and when `LYM_GATE_TEST_PID` names a live ancestor, since the outer guard already sees those binaries.
- `internal/gatecli`, tier runs: a tier with `memory_mb` runs its command under `proc.RunMemoryCappedVia`.
  The shell stays tied to the verb.
  A kill at that cap fails the run with a line naming the tier and its cap, and the run is never rerun.
- `internal/gateslot`: the new hub key `memory_quit_mb`, cross-pair, defaults to 4096, and 0 turns the guard off.
  The new tier key `memory_mb` is optional and at least 1.
  `template.yaml`, the package doc and `TestLoadConfig` carry both.
- `internal/proc`:
  - `TreeCPU`/`ProcessCPU` become `TreeSample`/`ProcessSample`, with each process's parent and resident memory, and `cpu.go` is now `sample.go`.
  - `ReadProcess` reads one process's links and command line; `ListProcesses` now uses it.
  - `RunMemoryCappedVia` runs a capped command through a caller's own start and wait, and reports a kill of any process of the tree at the cap.
    `RunMemoryCapped` now runs on the same internals, and its result is as before.
- `PATTERN-daemon-wakeups` and the wakeup scan's file list name the guard's one-second loop.
  `docs/overview.md` mentions the guard and `memory_mb`.

## Design decisions

- `memory_mb` has no default, so a tier runs uncapped unless the hub sets it.
  One cap holds the tier's builds and every test binary together, and the right size depends on the tier's parallel and packages.
  The guard is on by default, so every tier is guarded without one.
- The guard's per-binary cap, 4096 MB, sits above `redundancy_test_memory_mb`'s 2048 MB.
  So in `lym gate redundant` and `lym gate testmap` the kernel cap stays the hard bound.
  The guard catches what runs outside it, and names the culprit.
- The guard names the package by the binary's working directory, which `go test` sets to the package directory.
  It names the test only through `-test.run`.
  A tier run without `-run` gets the package and not the test, and the tier's failure report lists the package's failure.
- The pressure trigger spares a tree under 1024 MB, so a healthy small test is not failed for memory held elsewhere on the machine.
- The guard samples once a second, the wakeup floor, since a runaway grows by a gigabyte or more a second.
- The `lym gate test` redundancy report step runs no guard of its own.
  It runs inside the gate call's process, after the tiers, under the kernel cap.

## To check by hand

- Set `memory_mb` on this hub's tiers, if wanted, with `lym config gate`; this change sets no hub value.
- No test drives a real runaway through the guard or a tier's `memory_mb`.
  The guard's decisions are covered on a scripted sampler, and `RunMemoryCappedVia` on a real capped scope.
  A run with a test that allocates past the cap, watched with the slice's memory, would confirm both end to end.
- The Windows paths of `RunMemoryCappedVia` were compiled and vetted but not run on Windows.
  `TreeSample` is unsupported there, so the guard logs once and ends.
