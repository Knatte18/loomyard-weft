# Card 1 — proc-tied-child

**Create:**
- `internal/proc#Tied`
- `internal/proc#StartTied`
- `internal/proc#Tied.Record`
- `internal/proc#Tied.Wait`
- `internal/proc#Tied.Quit`
- `internal/proc#Tied.Kill`
- `internal/proc#TreeRecord`
- `internal/proc#KillTree`
- `internal/proc#SelfRecord`
- `internal/proc#HoldNamedJob`
- `internal/proc#TerminateNamedJob`
- `internal/proc#IsBreakawayRefused`
- `internal/proc/tied_test.go`
- `internal/proc/tied_linux_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Move the child-tie mechanism `lyx gate test` carries today in its `child_unix.go` and `child_windows.go` into `internal/proc`, where the loop (card 12) and the teardown (card 18) can share it, and add the tree kill and the Windows named-job identity the decision record's D2, D4 and D11 need.
`StartTied` starts `cmd` tied to the calling process's life: on Linux it sets `Setpgid` and `Pdeathsig: SIGKILL` and runs the start and the wait on one goroutine locked to its OS thread for the child's whole life, since the parent-death signal follows the spawning thread; on Windows it creates a kill-on-close job object, starts the child and assigns it to that job, which the OS closes when the holder dies.
Each child runs in its own process group whose id is the child's pid.
When the caller set no `cmd.Cancel`, `StartTied` installs one that calls `Kill`, so an `exec.CommandContext` child still dies on cancel.
`Tied.Record` returns the child's `TreeRecord`: its pid, its process-group id and its start time as `StartTime` reads it (empty on Windows).
The per-child job is unnamed, so no other process can reopen it and the record carries nothing of it.
`Tied.Wait` returns the child's `Wait` error; `Tied.Quit` sends SIGQUIT to the child alone on Linux (so a Go child's goroutine dump lands on its stderr) and is a no-op on Windows; `Tied.Kill` kills the whole tree: SIGKILL to the recorded process group on Linux, `TerminateJobObject` on Windows, falling back to the process alone when the group or job kill fails.
`KillTree` is the standalone tree kill a later process runs from a record it read from disk: on Linux it reads `/proc/<pid>/stat` for every live process, kills the group with SIGKILL only while some live process reports the recorded group id with a start time at or after the recorded one (Linux keeps a group id unassigned while any member lives, so a match is the step's own tree, and the gate holds after the group leader is dead), and reports whether anything was killed; on Windows it kills nothing and reports false, because the child's kill-on-close job sits inside its loop's named job and closed when the loop died, and a live loop's tree is ended through `TerminateNamedJob` on the loop's job instead.
`SelfRecord` returns the calling process's own pid and start time (start empty on Windows), the shape a pid file records.
`HoldNamedJob` creates, on Windows, a kill-on-close job object under `name`, assigns the calling process to it and returns the function that releases the handle; the name exists only while a handle is open, so opening it by name later reaches the live holder and its whole tree, never a reused pid.
On Linux it returns a no-op release.
`TerminateNamedJob` opens the job by name and terminates it, reporting false when no job of that name exists; on Linux it always reports false.
`IsBreakawayRefused` reports whether a `Start` error is Windows' `ERROR_ACCESS_DENIED` from a `CREATE_BREAKAWAY_FROM_JOB` the enclosing job forbids; it is always false on Linux.
The per-GOOS bodies live in `tied_linux.go`, `tied_windows.go` and `namedjob_windows.go` plus a `namedjob_other.go` stub, as unexported `…Platform` helpers behind the shared exported symbols in `tied.go`, so every exported glyph resolves uniquely; the helpers follow `configurePlatformGroupKill`'s existing shape and take no handle here.
`tied_test.go` pins, through the real spawns this package's tier allowlist admits, that `StartTied` returns a record whose group id equals the pid and that `Kill` ends a grandchild (a shell that backgrounds a sleeper and prints its pid, then waits), and that `IsBreakawayRefused` is false for a nil and an ordinary error.
`tied_linux_test.go` pins `KillTree` on Linux: a record whose group is live kills it and reports true; a record whose start time lies after every live member's kills nothing and reports false; `SelfRecord` reports the test process's pid and a non-empty start.
`go vet` on a Linux host skips the Windows bodies, so the card's `**Verify:**` type-checks them with `GOOS=windows`; they run only on a Windows machine.
`proc` cannot log, so each caller of `Kill`, `Quit`, `KillTree` and `TerminateNamedJob` logs the kill and its outcome itself.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/proc` from 12 to 18.

**ImpactSummary:** New exported surface in `internal/proc` only; nothing calls it yet, so no caller changes.

**Verify:** GOOS=windows go vet ./internal/proc
