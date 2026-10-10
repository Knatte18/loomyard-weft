# Card 2 — testtiming-resources

**Create:**
- `plan:cmd/testtiming#runResources` -> `func runResources(tags, pkgFlag string) error`
- `plan:cmd/testtiming#resourceRow` -> `type resourceRow struct`
- `plan:cmd/testtiming#measurePackageResources` -> `func measurePackageResources(tags, importPath, tmpDir string, env []string, parallel int) (resourceRow, error)`
- `plan:cmd/testtiming#readForkCount` -> `func readForkCount() (int64, bool)`
- `plan:cmd/testtiming#readLoadAverage` -> `func readLoadAverage() string`
- `plan:cmd/testtiming#parseForkCount` -> `func parseForkCount(stat string) (int64, bool)`
- `plan:cmd/testtiming#scopeStats` -> `func scopeStats(cgroupDir string) (cpu time.Duration, peakBytes int64, err error)`
- `plan:cmd/testtiming#parseCPUStat` -> `func parseCPUStat(stat string) (time.Duration, error)`
- `plan:cmd/testtiming#rusageStats` -> `func rusageStats(state *os.ProcessState) (cpu time.Duration, maxRSS int64)`
- `plan:cmd/testtiming#leftoverProcesses` -> `func leftoverProcesses(dir string) []string`
- `plan:cmd/testtiming#takeGateSlot` -> `func takeGateSlot(ctx context.Context, cwd string) (env []string, parallel int, release func(), err error)`
- `plan:cmd/testtiming#renderResources` -> `func renderResources(tier string, rows []resourceRow, loadBefore, loadAfter string, cgroups bool) string`
- `cmd/testtiming/resources_test.go`

**Edit:**
- `cmd/testtiming#main`
- `cmd/testtiming/main_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/hubgeom#GateSlots`
- `internal/gateslot#TemplateConfig`
- `internal/lyxcwd#ResolveWorktree`
- `internal/lyxcwd#Getwd`

**Intent:** The `-resources` mode of D1, in a new file `cmd/testtiming/resources.go`.
`main` gains the `-resources` flag; it keeps `-full`, `-tags`, `-top` and `-pkg`, and `-resources` together with `-redundancy` is refused with one message naming both flags, like `-full` with `-tags` is today.
`runResources` lists the tier's packages (`listPackages`), prints the one-minute load average from `/proc/loadavg` at start and at end, then runs the packages serially: each as its own `go test -json -count=1` child over one package, with `-tags` when given and `-p` capped.
`takeGateSlot` resolves the cwd through `lyxcwd.Getwd` and `lyxcwd.ResolveWorktree`; inside a hub (the board's `_lyx` present, as `preflight.BoardLyxPresent` decides) it acquires one slot of `hubgeom.GateSlots` for the whole run, writes no wait record, and returns the lease's environment and `GoParallel`; outside a hub it returns the template config's cap, as `lyx gate test` does.
`measurePackageResources` creates a private temp directory per package, points the child's `TMPDIR` (and `TMP`, `TEMP`) at it, and measures: wall time from the child's start to its exit; CPU and peak memory from the child's cgroup when `systemd-run` is on PATH, by running the child as `systemd-run --user --scope --quiet -- go test ...` and reading `cpu.stat`'s `usage_usec` and `memory.peak` from the scope directory named by `/proc/<pid>/cgroup`; otherwise the child's rusage (`ProcessState.SysUsage`) gives CPU and the largest single process's max RSS, and the row is marked `rusage`; process count as the delta of the `processes` line of `/proc/stat` across the child, blank when the file is unreadable; leftover processes as every `/proc` entry whose `cwd`, `exe` or an `argv` element references the package's temp directory after the child exits, listed as `pid argv`.
Pass or fail per package comes from the JSON stream through the existing `parseLine`.
`renderResources` prints one table per run with columns `PACKAGE`, `WALL`, `CPU`, `PEAK_MEM`, `PROCS`, `LEFTOVER`, a `FAIL` mark, and a trailer that names the CPU and memory source, the load averages and that the process count is system-wide, so a report reads "quiet machine" as a precondition.
Exit code mirrors `go test`, as `run` does.
The pure parsers (`parseForkCount`, `parseCPUStat`, the leftover matcher and `renderResources`) are tested in `resources_test.go`, untagged, over fixture text; the test file spells no `gitkit.` export, no `exec.Command` and no `lyxbin.` token, since the tier-purity scan reads it raw.
`main_test.go`'s flag-conflict rows gain the `-resources` with `-redundancy` refusal.
The package comment in `main.go` documents the mode, its columns, their sources and the Linux-first fallback.
`cmd/testtiming`'s budget row rises to cover the new test functions.

**ImpactSummary:** `main` grows one flag and one branch; `parseLine`, `listPackages` and `pkgResult` are reused unchanged, and no other package is affected.
