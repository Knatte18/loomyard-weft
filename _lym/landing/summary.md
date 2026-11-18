# Report a memory-cap kill reliably and refuse a scope not held to its cap

`proc.RunMemoryCappedVia` sometimes reported no kill at the cap when the kernel had killed the child at it.
This change makes that report reliable and turns a scope not held to its cap into an error.

## Root cause

The memory cap held in both failing gate runs (`20261118T131024Z-tier1-808563` and `20261118T131227Z-tier1-1218541`).
The test's hog prints `kept 384 megabytes` once it holds its memory, and the captured output was `survived\n` alone.
The kernel had killed the hog at the 128 MB cap before it got there.

The kill went unreported.
`runMemoryCapped` read the scope's `memory.events` from a goroutine that opened the file only once the scope's shell had written its cgroup.
When the command was a shell whose child the cap killed, the shell then exited and systemd removed the empty scope.
On a loaded host the goroutine could first run after that removal, and `scopeHadOOMKill` after the wait found the scope gone too.
Both read as no kill.
Only `TestRunMemoryCappedVia_StartsThroughItsStartAndReportsAChildKilledAtTheCap` asserts a tree kill for a command that survives its child,
which is why it alone failed.

The machine hang the same day was not a cap failure.
The journal's `memsample` lines show the 11.4 GB process was `tail -8`, the pipeline stage outside the scope of a hand-run probe.
The scope itself peaked at 3.9 MB.

## What changes

- **`internal/proc/memorycap_linux.go`**: the scope's start script now starts a holder process inside the scope before it becomes the command.
  The holder waits on a pipe whose write end the caller keeps, so the scope and its `memory.events` outlast the command.
  The caller reads the events once the command has ended, then closes the pipe on every return, which ends the holder.
  The watch goroutine and its `poll` of `memory.events` are gone.
  The read happens after the scope has done all its work, and it can no longer race the scope's removal.
- The holder leaves the shell's tree at once through a subshell, so the command never has it as a child.
  The subshell points its stdin, stdout and stderr at `/dev/null` first:
  dash keeps the shell's stdout on a background builtin despite the builtin's own redirection,
  and a holder on the command's stdout pipe would keep `cmd.Wait` waiting for output forever.
- **A scope not held to the cap never runs the command.**
  The start script reads its own cgroup's `memory.max` and compares it with the limit rounded down to a whole page.
  On a mismatch, or when the file cannot be read, it names both values on stderr, exits 125 and writes no cgroup.
  The call then returns an error wrapping `ErrMemoryCap` that names the expected `memory.max`.
  A user manager that accepts `MemoryMax` and ignores it can no longer run a capped command silently uncapped.
- `internal/proc/doc.go` states the check and the read after the command's end.

## Tests

- The tree-kill test gains a row whose start returns only once the command has ended.
  It stands in for a caller slow to read the scope, which still sees the kill.
- The scope-start test becomes `TestRunMemoryCapped_AScopeNotHeldToTheCapIsAnErrorAndNeverRunsTheCommand`.
  It gains a row whose stand-in `systemd-run` runs the command in the test's own cgroup, as a user manager that ignores the cap does.
  The command must not run, and the error must wrap `ErrMemoryCap`.
  The stand-in is written before the test goes parallel, so a parallel fork cannot inherit its open write and fail the exec with ETXTBSY.
- Every hog is bounded: it keeps a fixed 384 MB and exits.

`lym gate test --tier 1 ./internal/proc -- -count=50 -run TestRunMemoryCapped` passed 50 runs in a row
(800 results, 0 failed, 15 s wall), with tier-1 gate runs of other packages beside it
(log `_bolt/.lym/gate/runs/20261118T140719Z-tier1-124652.log`).

## Design decisions

- The holder keeps the scope rather than wrapping the command in a shell that reports the count on exit,
  so the command keeps its own pid, process group and exit status, which cancels and kill reports depend on.
- The `memory.max` check runs inside the scope before the command, not in Go after it,
  so a cap that is not in force stops the command before it allocates anything.
