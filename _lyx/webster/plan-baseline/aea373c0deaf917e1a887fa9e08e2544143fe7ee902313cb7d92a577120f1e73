# Card 2 — proc-creation-time

**Create:**
- `internal/proc#CreationTime`
- `internal/proc#TestCreationTime`

**Edit:**
- `internal/proc/doc.go`

**Intent:**
Add the Windows read of a process's creation time by pid that D4's reused-pid filter needs.
On Windows, `CreationTime` opens the process with `PROCESS_QUERY_LIMITED_INFORMATION` through `golang.org/x/sys/windows` (already a module dependency, used by `tied_windows.go`), reads `GetProcessTimes`, closes the handle and returns the creation time as a `time.Time` and true; a pid it cannot open or read returns the zero time and false.
On every other platform it returns the zero time and false, since D4's filter applies only where `runproc` keeps records with an empty start time; the Windows body goes in `proc_windows.go` and the other in a `//go:build !windows` file.
`TestCreationTime` is one test in a file with no build tag: on Windows it asserts the calling process's creation time is found, not after `time.Now()`, and that a pid no process holds reads false; elsewhere it asserts the calling process reads false.
`doc.go` names `CreationTime` and its per-platform answer in the paragraph that lists what works where.

**ImpactSummary:** One new exported function in `proc` and a doc line; no existing caller changes.
