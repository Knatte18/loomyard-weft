# Gate memory guard and per-tier kernel memory cap

A test that allocates without bound in a gate run could freeze the machine.
The gate watchdog watched CPU only, and the gate tiers' `go test` runs had no memory cap.
This change adds a memory guard over every test binary lym starts, and an optional kernel cap per gate tier.

## What changes

- **Memory guard** (`internal/gatecli/memoryguard.go`):
  while a tier's command runs, it samples the child tree once a second through `proc.TreeSample`.
  - A `.test` process whose tree holds more than `memory_quit_mb` resident is killed whole with SIGKILL.
    A tree that holds another tree over the cap is spared, and the inner binary is killed.
  - When no tree passes the cap, the guard reads the memory limit lym sits under (`proc.ReadMemoryLimit`).
    With less than a tenth of its Max left, it kills the largest tree of 1024 MB or more.
  - Each kill prints a line naming the binary, its package directory, the `-test.run` value it was given and the reason.
    The tier's log and failure report carry the line, and the tier is never rerun.
    A go-json tier also names the tests that had started and never passed, failed or skipped.
  - `lym gate redundant`, `lym gate testmap` and the redundancy report step of `lym gate test` run the same guard.
    A test run it kills there is not judged, a killed mutant run counts as no mutant kill,
    and `lym gate testmap` records the entry as failed.
  - Off Linux the guard logs that it cannot sample, and ends.
  - It is off when `memory_quit_mb` is 0, or when `LYM_GATE_TEST_PID` names a live ancestor whose own guard sees these binaries.
- **Config key `memory_quit_mb`** (`internal/gateslot`):
  a cross-pair key with the template default 4096, validated to at least 0.
- **Per-tier `memory_mb`** (`internal/gateslot/tiers.go`, `internal/gatecli/tierrun.go`):
  a tier that sets it runs its command under the kernel cap, covering its builds and test binaries together.
  A kill at the cap fails the run with a line naming the tier and its cap, and the tier is never rerun.
  Where the user manager cannot cap a scope, the cap falls back to each process's data segment.
  A command that cannot start under the cap is a refusal naming the cap and its way forward.
- **`internal/proc`**:
  - `TreeCPU` becomes `TreeSample`, which also reports each process's parent and resident memory; `ReadProcess` reads one process.
  - `RunMemoryCappedVia` runs a command under the cap through the caller's own start and wait,
    so the tier shell stays tied to the verb.
    It reports whether the kernel killed any process of the tree.
  - `ErrMemoryCap` marks a cap that could not be set, as opposed to a verdict on the command.
  - On Linux the scope's `memory.events` is watched by `poll` with no timeout, so the watch wakes only when the events change.
    A kill at an enclosing cgroup's limit is not reported as a kill at the scope's.
  - After a SIGKILL of the command, whatever is left in the scope is killed through `cgroup.kill`.
- `internal/testcover` and the redundancy verbs pass the guard's verdicts through.
  `pattern/PATTERN-daemon-wakeups.md` lists the guard's one-second sampling.

## Design decisions

- A tier is uncapped unless the hub sets `memory_mb` on it.
  The change ships no default and sets no hub value.
  A tier's cap bounds its builds and test binaries together, and the need differs per hub and machine,
  so a default would either fail healthy runs or bound nothing.
  The guard's pressure trigger is the default bound on every tier run.
  The operator sets each hub's value from measured peaks after this lands.
- The guard watches `.test` processes only, so a `vstest` tier is bounded by `timeout_min` and, if set, `memory_mb`.
- The guard sits beside the `redundancy_test_memory_mb` kernel cap rather than replacing it:
  the kernel cap is the hard bound, and the guard names the culprit.

## To check by hand

Nothing ran beyond the unit tests, and no regression check ran for this task.
Check these open findings:

- A description or doc claim that contradicts the diff (`change-description-contradicts-diff`):
  re-read the guard and cap paragraphs of `internal/gatecli/doc.go` against the code.
- A guard kill of an inner test binary leaves the outer binary's pid unnamed in the kill line
  (`guard-kill-of-inner-binary-leaves-outer-pid-unnamed`, `internal/gatecli/memoryguard.go`).
- The OOM watch in `internal/proc/memorycap_linux.go`
  (`oom-poll-wakes-every-20ms`): confirm the poll now wakes only on `memory.events` changes.
- The scope-start read deadline in `internal/proc/memorycap_linux.go` can misread a slow start as a failed one
  (`scope-start-read-deadline-misreads-slow-start`).
- On a Linux host with a working user manager, run a tier with `memory_mb` set low and a test that allocates past it,
  and confirm the failure line names the tier and cap.
