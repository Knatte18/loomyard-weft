# Card 8 — tmuxkit-proc-probes

**Create:**
- `plan:internal/testkit/tmuxkit#procStatFields` -> `func procStatFields(pid int) ([]string, bool)`
- `plan:internal/testkit/tmuxkit#ProcPPID` -> `func ProcPPID(pid int) (int, bool)`
- `plan:internal/testkit/tmuxkit#ProcState` -> `func ProcState(pid int) (string, bool)`
- `plan:internal/testkit/tmuxkit#ProcComm` -> `func ProcComm(pid int) (string, bool)`
- `plan:internal/testkit/tmuxkit#TestProcProbes_OwnProcess` -> `func TestProcProbes_OwnProcess(t *testing.T)`

**Edit:**
- `internal/testkit/tmuxkit/proc.go`
- `internal/testkit/tmuxkit/tmuxkit.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Give `tmuxkit` the three read-only probes of decision D8, in `proc.go` beside the existing ones, each following their shape: nothing on a missing or unreadable process, nothing where `/proc` does not exist, and no process started.
`procStatFields` reads `/proc/<pid>/stat` and returns the fields after the last `)`, the comm-safe parse `procSession` does today; `procSession` and the two stat probes share it.
`ProcPPID` returns field 1 of those, the parent pid; `ProcState` returns field 0, the state letter; `ProcComm` reads `/proc/<pid>/comm` trimmed of its newline.
`proc.go`'s header comment lists the new probes.
`TestProcProbes_OwnProcess`, in a new untagged `internal/testkit/tmuxkit/proc_test.go`, skips where `runtime.GOOS` is not `linux`, and otherwise checks the probes against the test's own process: `ProcPPID` of `os.Getpid()` equals `os.Getppid()`, `ProcState` answers one of `R`, `S` or `D`, `ProcComm` equals the base name of `os.Args[0]` truncated to the kernel's fifteen bytes, and all three answer false for a pid above the kernel's pid range, such as `1 << 30`.
It calls `t.Parallel` and spawns nothing, so it is Tier 1.
`tmuxkit.go`'s package doc lists the three probes beside the existing five in its sentence on the exported `/proc` probes.
The kit's `os/exec` exemption bounds are unchanged, since the probes read files only.
The `internal/testkit/tmuxkit` row of `cmd/lyx/testdata/test-budget.yaml` rises by one.

**ImpactSummary:** Three new kit exports and one shared parse; procSession keeps its contract, and no caller outside the kit changes until card 9.
