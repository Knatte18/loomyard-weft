# Card 11 — launch prompt parks at a hand-back

**Edit:**
- `internal/loomcli/driverprompt.go`
- `internal/loomcli/driverprompt_test.go`

**Uses:**
- `internal/loomcli#driverTeardownCommand`

**Intent:**
The loom-launched driver's launch prompt stops ending the session at every stop (decision record D6).

`driverPrompt` keeps naming the skill, the run-id, the report path, autonomous mode and the missing-skill instruction.
Its closing instructions change:
- at `done`, and at a `busy` refusal, the driver runs `driverTeardownCommand` as its very last act, exactly as today;
- at every other stop, it parks as the ly-drive skill describes, leaving this session open, and `lyx loom start` resumes it by typing one line.
The prompt says "park" in words the skill's parking section keys on, so a session launched without this prompt (an orchestrator fork, an operator-launched session) never parks.
`driverTeardownCommand` and its composition are unchanged.
The prompt stays well under `maxLaunchPromptBytes`.
`driverPrompt`'s doc comment covers the park/teardown split.
It also states the batten half of the decision record's end-to-end acceptance criterion this split keeps: batten's Inner-Run row, on a `done` child, waits for the driver strand to end before it returns `Done` and teardown runs (`callDone`), and the teardown at `done` is what ends that strand.

Tests (`driverprompt_test.go`):
- The prompt still names the run-id, the report path and autonomous mode, and still contains `driverTeardownCommand` exactly once.
- The prompt ties the teardown command to `done` and `busy` and names parking for every other stop, so a `done` child's driver still exits before batten's teardown.
- `TestDriverPrompt_StaysWellUnderLaunchPromptCap` keeps passing for a realistic run-id and report path.

**ImpactSummary:** Changes the launch prompt text of every newly spawned llm driver; the Go driver, the teardown command and every other caller are unchanged.

**Commit:** 11: loomcli: the ly-drive launch prompt parks at a hand-back
