# proc: hold a memory-capped scope until its kill count is read

`RunMemoryCapped` and `RunMemoryCappedVia` sometimes reported no kill for a child that kept 384 MB under a 128 MB cap.
`TestRunMemoryCappedVia_StartsThroughItsStartAndReportsAChildKilledAtTheCap` failed intermittently with `tree killed false, <nil>, 1 starts, output "survived\n"`.

## Root cause

The kill at the cap is read from the scope's `memory.events`.
The kernel removes a scope with its last process, and the count goes with it.
The old code polled `memory.events` while the command ran and read it once more after the command ended.
When the kill was not seen in a poll, the scope was often already gone at the final read.
The call then reported a plain exit, so a capped command that the kernel had killed looked like one that stayed within the cap.

## What changes

- A holder process now keeps the scope alive until the caller has read `memory.events`.
  The scope's start script launches it in a session of its own with stdin, stdout and stderr on `/dev/null`.
  It is therefore never a child of the command, never in the command's process group, and holds no pipe of the command's.
  It ends when the caller closes the write end of a pipe, which happens on every return of `runMemoryCapped`.
- `watchForOOMKill` is removed.
  With the scope held, one read of `memory.events` after the command has ended is enough, so no poll runs beside the command.
- The start script checks the scope's own `memory.max` against the cap before it runs the command.
  The cap is the limit rounded down to a whole page, as the kernel stores it.
  A scope with another value, or whose `PATH` has no `setsid` for the holder, exits 125 and never runs the command.
  The call returns an error wrapping `ErrMemoryCap`, so a capped command can no longer run uncapped without saying so.
- `cgroupMemoryMaxFile` names the `memory.max` file once for `memorycap_linux.go` and `memorylimit.go`.

## Tests

- The kill test now runs with a start that returns at once and with one that returns only once the command has ended, which stands in for a caller slow to read the scope.
- New tests pin that the scope is removed after a capped run with no kill, that the holder is no child of the command, and that a scope not held to the cap or without `setsid` is an `ErrMemoryCap` error that never runs the command.
- A cap that is no multiple of the page size is covered by a within-cap child that must still run.
- The cap test in `internal/gatecli` no longer runs an unbounded hog.
  Its command is `head -c 268435456 /dev/zero | tail`, so a cap that fails to hold costs 256 MB and never the machine.
  The background `sleep` that kept the scope alive is gone, as the holder does that now.

## To check by hand

- Run the `internal/proc` memory cap tests many times in a row, also with other gate work running beside them, and state the count.
  This was not run for this description.
- `holder-test-repeats-cgroup-path-literals`, in `TestRunMemoryCapped_TheHolderEndsOnceACappedRunWithNoKillReturns` in `internal/proc/memorycap_linux_test.go`:
  the diff reads the cgroup paths from `proc.SelfCgroupFile` and `proc.CgroupMount`, so confirm no literal is left.
- `kept-tests-missing-testtiming-keep`, in `internal/proc/memorycap_linux_test.go`:
  the diff marks the holder-end test and the no-`setsid` test with `testtiming:keep`, so confirm each carries it with a reason that holds.
