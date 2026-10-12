# Card 9 — Sample a process tree's CPU time and walk ancestry in proc

**Create:**
- `internal/proc/cpu.go`
- `internal/proc/doc.go`
- `plan:internal/proc#ProcessCPU` -> `type ProcessCPU struct`
- `plan:internal/proc#TreeCPU` -> `func TreeCPU(root int) ([]ProcessCPU, error)`
- `plan:internal/proc#IsAncestor` -> `func IsAncestor(pid int) bool`
- `plan:internal/proc#TestProcessTreeReads` -> `func TestProcessTreeReads(t *testing.T)`

**Edit:**
- `internal/proc/proc_linux.go`
- `internal/proc/proc_windows.go`
- `internal/proc/proc_linux_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D7's sampling and D8's platform split, in `internal/proc` beside its existing `/proc/<pid>/stat` reader; the watchdog loop that uses them is card 10's.

- `cpu.go`, with no build constraint, declares `ProcessCPU`: `PID int`, `Path string`, the executable path, and `CPU time.Duration`, the user plus system time the process has used.
- `TreeCPU(root)`, in `proc_linux.go`, lists `/proc`, reads each numeric entry's stat through `statFieldsAfterName` for its parent pid (field 4) and its user and system ticks (fields 14 and 15), and returns `root` and every descendant of it.
  Ticks convert at the kernel's fixed `USER_HZ` of 100 per second, a named constant.
  `Path` comes from `os.Readlink("/proc/<pid>/exe")`, falling back to the first NUL-separated field of `/proc/<pid>/cmdline`, never from the 15-character `comm` name in `stat`, so a test binary whose name is longer than 15 characters keeps its `.test` suffix.
  A process that ends between the listing and its reads is skipped; an unreadable `/proc` is an error.
- `IsAncestor(pid)`, in `proc_linux.go`, walks the parent pids from `os.Getppid()` upward through `statFieldsAfterName` and reports whether it meets `pid`; the process itself is not its own ancestor, and pid 0 or a pid that is not alive is never one.
- `proc_windows.go` gains the two stubs: `TreeCPU` returns `nil, errors.ErrUnsupported`, the error card 10's watchdog reads as "no watchdog applies", and `IsAncestor` returns false.
- The package doc moves from `proc_windows.go`, which builds only on Windows, into the new `doc.go`, and gains the CPU sampling and ancestry reads and their Linux-only scope.

`TestProcessTreeReads`, in `proc_linux_test.go`, starts a `sleep 30` child, killed in cleanup, and burns about 100 ms of CPU in the test goroutine, then asserts:
- `TreeCPU(os.Getpid())` holds the test process with `Path` equal to `os.Executable()` and a `CPU` above zero, and holds the child's pid with a non-empty `Path`, since where `sleep` lives depends on the host's coreutils;
- `TreeCPU(childPID)` holds the child and not the test process;
- `IsAncestor(os.Getppid())` is true, while `IsAncestor(os.Getpid())`, `IsAncestor(childPID)` and `IsAncestor` of a pid past the kernel's maximum are false.
`internal/proc`'s budget row rises from 17 to 18.

**ImpactSummary:** two new exported reads and one type in `internal/proc`, with no existing caller changed; the package doc moves to a file every platform builds.

**Commit:** 9: proc reads a process tree's CPU time and its ancestry
