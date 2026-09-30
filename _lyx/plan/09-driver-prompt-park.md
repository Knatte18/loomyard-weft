# Card 9 — launch prompt parks at a hand-back; resume line

**Create:**
- `plan:internal/loomcli#driverResumeLine` -> `func driverResumeLine(runID string, reportPath string) string`

**Edit:**
- `internal/loomcli/driverprompt.go`
- `internal/loomcli/driverprompt_test.go`

**Uses:**
- `internal/loomcli#driverTeardownCommand`

**Intent:**
The loom-launched driver's launch prompt stops ending the session at every stop (decision record D6), and loom gains the one line it types to resume a parked driver (D8).

`driverPrompt` keeps naming the skill, the run-id, the report path, autonomous mode and the missing-skill instruction.
Its closing instructions change:
- at `done`, and at a `busy` refusal, the driver runs `driverTeardownCommand` as its very last act, exactly as today;
- at every other stop, it parks as the ly-drive skill describes, leaving this session open, and `lyx loom start` resumes it by typing one line.
The prompt says "park" in words the skill's parking section keys on, so a session launched without this prompt (an orchestrator fork, an operator-launched session) never parks.
`driverTeardownCommand` and its composition are unchanged.
The prompt stays well under `maxLaunchPromptBytes`.

`driverResumeLine` composes the single line `lyx loom start` sends a parked driver.
It names the ly-drive run by run-id and tells the driver to:
- take a fresh baseline;
- reset its repair and re-step budgets;
- write its next stop report to the given path.
It carries no newline, so `shuttleengine.Runner.Send` accepts it, and it composes prompt text only, naming no provider flag or command line, per the Shuttle Provider-Seam Invariant.
The doc comments of both functions cover the park/teardown split and state that a resume line arriving during an in-flight step starts no second step (the skill owns that rule; the line only carries the path and the reset).

Tests (`driverprompt_test.go`):
- The prompt still names the run-id, the report path and autonomous mode, and still contains `driverTeardownCommand` exactly once.
- The prompt ties the teardown command to `done` and `busy` and names parking for every other stop.
- `TestDriverPrompt_StaysWellUnderLaunchPromptCap` keeps passing for a realistic run-id and report path.
- `driverResumeLine` contains the run-id and the report path, and the strings `baseline` and `budget`.
  It contains no `\n` or `\r` and is non-empty.

**ImpactSummary:** Changes the launch prompt text of every newly spawned llm driver and adds one unexported composer; the Go driver, the teardown command and every other caller are unchanged.

**Commit:** 9: loomcli: the ly-drive launch prompt parks at a hand-back; add the resume line
