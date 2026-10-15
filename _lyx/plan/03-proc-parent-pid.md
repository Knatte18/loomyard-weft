# Card 3 — proc-parent-pid

**Create:**
- `internal/proc#ParentPID`

**Edit:**
- `internal/proc#TestProcessTreeReads`

**Intent:** Add the one process read creel's sender identity walk needs (D4): the parent of an arbitrary pid.
On Linux, in `internal/proc/proc_linux.go`, `ParentPID` reads field 4 of `/proc/<pid>/stat` through `statFieldsAfterName` (the second field after the command name) and reports false for a pid that does not exist, a stat that does not parse and a pid of 0 or below.
On Windows, in `internal/proc/proc_windows.go`, it reports 0 and false, since creel resolves identity from process ancestry on Linux only.
`TestProcessTreeReads` gains assertions that `ParentPID` of its started `sleep` child is the test process's own pid, that `ParentPID(os.Getpid())` equals `os.Getppid()`, that `ParentPID(0)` reports false, and that the pid of its `sleep` child, once the child has been waited on, reports false.

**ImpactSummary:** Additive per-platform function in proc and one extended Linux test; no existing caller changes.
