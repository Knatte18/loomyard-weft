# Card 10 — The loom driver launches from a driver stencil

**Create:**
- `contracts/stencils/shed/shed-template-driver.md`

**Edit:**
- `contracts/stencils/stencils.go`
- `internal/loomcli/driverprompt.go`
- `internal/loomcli/driverspec.go`
- `internal/loomcli/start.go`
- `internal/loomcli/driverprompt_test.go`
- `internal/loomcli/driverspec_test.go`
- `internal/loomcli/start_driver_test.go`

**Rename:**
- `internal/loomcli/driveskill_test.go` -> `internal/loomcli/driverstencil_test.go`
- `cmd/lyx/lydriveskill_test.go` -> `cmd/lyx/driverstencil_test.go`

**Uses:**
- `plan:internal/parentdirective#Directive`
- `plan:internal/parentdirective#MarkerName`
- `internal/stencilstore#Read`

**Intent:**
D8: the driver's whole procedure becomes a stencil rendered as the loom driver's launch prompt, so the driver no longer depends on a skill being installed.

`shed-template-driver.md`, in a new `shed` family registered in `stencils.go`, carries the `ly-drive` skill's procedure (`plugins/ly/skills/ly-drive/SKILL.md`) for the one remaining mode, the loom-launched driver that always parks at a hand-back:
what it does, the drive directory (the session's own cwd), how to invoke a step, baseline, the loop, stopping, error envelopes, automatic re-steps, parking, binary watch, resume, interrupted invocations, repairing, the stranded-branch exception, the repair cap, ways forward (pointing at `contracts/specs/refusal-spec.md`, never restating it), repair records and the stop report, notifying the parent with the `parent_notice` relay, and filing nothing.
The "Driven from an orchestrator" section, the fork mode of the drive directory and the "Operator choices" section go; "Autonomous mode" becomes one sentence, since the driver is always autonomous.
Its markers are `{{.run_id}}`, `{{.report_path}}`, `{{.park_command}}`, `{{.teardown_command}}` and `{{.parent_directive}}`; the parent notification addresses the parent the directive names, and with no parent the stop report alone is the escalation.
It stays recipe-blind: its body names no recipe, no `.lyx/` path and no `lyx loom` or `lyx batten` command, which arrive only through the markers.

`driverPrompt` becomes `driverPrompt(stencilsDir, parentName, runID, reportPath string) (string, error)`: it reads the stencil through `stencilstore.Read` and fills it with the run-id, the report path, `driverParkCommand(reportPath)`, `driverTeardownCommand` and `parentdirective.Directive(stencilsDir, parentName, false)`.
The launch prompt is now the stencil, not a pointer at a skill, so the "skill not available" fallback sentence goes.
`startLLMDriverArm` passes the stencils directory and the receiver's `parentName` (card 4), and refuses with the render error when the stencil cannot be read.
`driverSpec` sets `Skills` to the driver's D2 list, `scribe:prose`, declared beside `driverRole`.

The two pin files move with `git mv` and read the embedded `stencils.ShedTemplateDriver` instead of `SKILL.md`; their tests are renamed from `DriveSkill`/`LyDriveSkill` to `DriverStencil`.
The recipe-blind scan and the park-marker, files-nothing, `friction: failed`, `parent_notice` and no-loom-row pins all run against the stencil body.
`driverprompt_test.go` checks the rendered prompt carries every marker's value and the directive; `driverspec_test.go` checks the skill list; `start_driver_test.go`'s prompt assertions follow the new composer.

**ImpactSummary:** the driver's launch prompt and spec in loomcli plus two moved pin files; the skill itself is still installed and removed in card 11.

**Commit:** 10: loom: the driver launches from the shed driver stencil
