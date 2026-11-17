# gate redundant: cap each test run's memory against runaway mutants

`lym gate redundant` runs mutant test binaries four at once with no memory bound.
A runaway mutant in `internal/websterengine` grew about 1–1.5 GB/s, pushed the user slice past its `MemoryHigh` throttle, and stalled the whole desktop, lym included, before the 10 s mutant timeout fired.
Neve froze twice that way.
This change caps the memory of every test-binary run, so the kernel ends the runaway and nothing else.

## What changes

- `internal/testcover`: `Toolchain` gains `MemoryLimit`, applied to every test-binary run at its choke point.
  `RunResult` gains `MemoryCapped`, set beside `TimedOut` for a run ended at the cap.
  `RunAlone` returns such a run as failed with `MemoryCapped`, never as an error.
- `internal/proc`: new `RunMemoryCapped`, which runs a command with the memory of its whole process tree capped.
  - Linux with a user systemd manager: the command runs in a transient scope with `MemoryMax=<cap>` and no swap.
  - Linux without one: each process gets `RLIMIT_DATA` at the cap, and the caller runs one capped command at a time.
  - Windows: a job object with a job memory limit, which also kills the tree.
  - `ReadMemoryLimit` reads the tightest `memory.high` or `memory.max` up the cgroup ancestry, for the clamp below.
- `internal/gateslot`: new key `redundancy_test_memory_mb`, default 2048 (unmutated runs sit near 300 MB).
  It is in `template.yaml`, the config docs and `TestLoadConfig`.
- `internal/gatecli`, `lym gate redundant`:
  - A capped mutant run counts as a kill, as a timeout does, and is never rerun alone.
  - A capped unmutated run sets `FailsUnmutated`.
  - The answer lists the timed-out and the capped mutants (file, line, operator, test).
  - The mutant timeout floor drops from 10 s to 2 s.
    The survey's `internal/loomcli` run spent 90 % of 542 s on workers waiting out 10 s timeouts.
- `lym gate testmap` and the redundancy report step share `RunAlone`, so the cap covers them.
  `testmap` also clamps its side-by-side runs to the memory headroom.
- `cmd/testtiming/redundancy.go`, which ran with no timeout, now runs each test under a 2-minute timeout and the 2048 MB cap.

## Design decisions

- Parallel is clamped to the memory headroom divided by the cap (at least 1), so the caps together stay under the slice's limit.
  Without a user scope manager the clamp is 1.
- The fallback uses `RLIMIT_DATA`, not `RLIMIT_AS`.
  Go reserves far more address space than it uses, so a 2048 MB `RLIMIT_AS` fails healthy Go test binaries at start-up.
  Since Linux 4.7 `RLIMIT_DATA` counts private anonymous mappings, where Go's heap lives.
- `GOMEMLIMIT` alone is no fix: it is a soft limit.
- Every allocating test fixture stops itself at a few hundred MB, so a broken cap fails the test and never the machine.
  The change was verified only on such fixtures; `lym gate redundant` and `lym gate testmap` were not run on a real package.

## To check by hand

- `lym gate redundant` on `internal/websterengine` was not run, as the task's bounds forbid it.
  Run it once, watching memory, to confirm the runaway mutant now ends at the cap.
- Windows paths (`memorycap_windows.go`, `namedjob_windows.go`) were not run on Windows here.
- `memory-scope-probe-unbounded` (`internal/proc/memorycap_linux.go`): the probe for a working user scope is unbounded in time.
- `scope-never-starts-reads-as-failing-test` (`internal/proc/memorycap_linux.go`, `internal/testcover/runner.go`): a scope that never starts can read as a failing test.
- `runalone-oom-branch-untested-on-scope-host` (`internal/testcover/runner.go`): the out-of-memory-line branch of `RunAlone` is not exercised on a host that has a scope.
- `report-step-and-testmap-capped-paths-untested` (`internal/gatecli/testmap.go`, `internal/gatecli/redundancy.go`): the capped paths of the report step and `testmap` have thin test coverage.
- `testmap-capped-runs-lack-parallel-bound` (`internal/gatecli/testmap.go`): check that `testmap`'s capped runs are bounded in parallel on every host.
- `rlimit-data-replaces-rlimit-as-unrecorded`: the `RLIMIT_DATA` choice is recorded above; confirm it matches the intent.
