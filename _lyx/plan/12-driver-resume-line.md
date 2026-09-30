# Card 12 — compose the resume line

**Create:**
- `internal/loomcli#driverResumeLine`
- `internal/loomcli/driverresumeline.go`
- `internal/loomcli/driverresumeline_test.go`

**Intent:**
Compose the one line `lyx loom start` types to resume a parked driver (decision record D8).

`driverResumeLine` names the ly-drive run by run-id and tells the driver to:
- take a fresh baseline;
- reset its repair and re-step budgets;
- write its next stop report to the given path.
It carries no newline, so `shuttleengine.Runner.Send` accepts it, and it composes prompt text only, naming no provider flag or command line, per the Shuttle Provider-Seam Invariant.
Its doc comment states that a resume line arriving during an in-flight step starts no second step: the skill owns that rule, and the line only carries the path and the reset.

Tests (`driverresumeline_test.go`):
- `driverResumeLine` contains the run-id and the report path, and the strings `baseline` and `budget`.
- It contains no `\n` or `\r` and is non-empty.

**Commit:** 12: loomcli: compose the parked-driver resume line
