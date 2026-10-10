# Card 19 — plan-gate-findings-file

**Create:**
- `internal/loomshed#writeInformationalFindings`

**Edit:**
- `internal/loomshed#NewPlanGate` -> `func NewPlanGate(anchorPath, worktreeRoot, scratchDir string, index planindex.Index) shuttleengine.Gate`
- `internal/loomshed#NewReworkPlanGate` -> `func NewReworkPlanGate(anchorPath, worktreeRoot, scratchDir string, index planindex.Index, readCommitted func(anchorRel string) ([]byte, bool, error)) shuttleengine.Gate`
- `internal/loomshed#planGate` -> `func planGate(gateName, anchorPath, scratchDir string, validate func(*planparser.Plan) ([]planindex.Finding, error)) shuttleengine.Gate`
- `internal/loomshed/gates_test.go`
- `internal/loomshed/gatefindings_test.go`
- `internal/loomshed/doc.go`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli/parity_test.go`
- `internal/loomcli/validate_test.go`

**Uses:**
- `internal/loomshed#formatPlanFindings`
- `internal/loomshed#hasBlockingFinding`
- `internal/shedrun#ScratchDir`

**Intent:** D10's first half: the plan gate's informational findings leave the trace as a count plus a file under the run's scratch directory instead of the whole formatted list.
The run's scratch directory is told, never derived: `shedrecipe.Env` gains `RunScratchDir`, the run's ephemeral shed scratch directory, which loom's `wire` fills with `shedrun.ScratchDir(location, c.runID)`, the same directory the step envelope reports as `scratch_dir`; `resolvePlainGateClosure` passes it to `NewPlanGate` and `NewReworkPlanGate`, which pass it to `planGate`.
Webster's own scratch directory is not used, because it is not the run's and the stop envelope never names it.
On the informational pass path `planGate` calls `writeInformationalFindings`, which writes `formatPlanFindings(findings)` one finding per line to `<gateName>-informational-findings.txt` under the told directory and returns the path; the Warn then carries the gate name, the plan directory, the count and that path, never the findings text.
An empty told directory or a write failure falls back to today's Warn with the findings inline, so nothing is lost and a caller with no run, such as a standalone validate, keeps today's behavior.
The blocking path is unchanged.
The test callers of the two constructors pass the new argument: `gates_test.go`, `gatefindings_test.go`, `parity_test.go` and `validate_test.go`, the last two with the directory `c.env` carries.
`gates_test.go`'s plan-gate informational case asserts the log carries the count and the path and that the file holds the finding; rows cover the unwritable-directory and empty-directory fallbacks.
`doc.go` describes the informational record.

**ImpactSummary:** Two exported constructors and `planGate` gain one argument, with their one production caller in `shedrecipe` and four test files updated; one log line's shape changes and one file lands under the run's ephemeral scratch directory; the gate's verdicts are unchanged.
