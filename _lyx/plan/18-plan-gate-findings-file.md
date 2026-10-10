# Card 18 — plan-gate-findings-file

**Create:**
- `plan:internal/loomshed#writeInformationalFindings` -> `func writeInformationalFindings(dir, gateName string, findings []planindex.Finding) (string, error)`

**Edit:**
- `internal/loomshed#planGate`
- `internal/loomshed/gates_test.go`
- `internal/loomshed/doc.go`

**Uses:**
- `internal/loomshed#formatPlanFindings`
- `internal/loomshed#hasBlockingFinding`

**Intent:** D10's first half: the plan gate's informational findings leave the trace as a count plus a file instead of the whole formatted list.
On the informational pass path `planGate` calls `writeInformationalFindings`, which writes `formatPlanFindings(findings)` one finding per line to `<gateName>-informational-findings.txt` under webster's scratch directory for the anchor, `websterengine.ScratchDir(anchorPath)`, the run's ephemeral directory the gate already reads its state from, and returns the path; the Warn then carries the gate name, the plan directory, the count and that path, never the findings text.
A write failure falls back to today's Warn with the findings inline, so nothing is lost.
The blocking path is unchanged.
`gates_test.go`'s plan-gate informational case asserts the log carries the count and the path and that the file holds the finding; a row covers the unwritable-directory fallback.
`doc.go` describes the informational record.

**ImpactSummary:** One log line's shape and one new file under an ephemeral directory; the gate's verdicts are unchanged.
