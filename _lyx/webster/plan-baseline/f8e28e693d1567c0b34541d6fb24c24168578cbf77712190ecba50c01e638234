# Card 17 — driver-stencil-rewrite

**Create:**
- `contracts/stencils/shed/shed-template-driver-guide.md`
- `contracts/stencils#ShedTemplateDriverGuide`
- `internal/loomcli#driverGuideStencilName`
- `internal/loomcli#driverGuideMarker`

**Edit:**
- `contracts/stencils/shed/shed-template-driver.md`
- `contracts/stencils#entries`
- `contracts/stencils#wordingClaims`
- `contracts/stencils/orch/orch-template-role.md`
- `internal/loomcli#driverPrompt`
- `internal/loomcli#driverSkills`
- `internal/loomcli#driverSpec`
- `internal/loomcli/driverprompt_test.go`
- `internal/loomcli/driverspec_test.go`
- `docs/overview.md`

**Uses:**
- `internal/stencilstore#Path`
- `internal/shedrun#ParkMarkerFileName`

**Intent:** D9: the driver stencil is rewritten around the loop, the heavy material moves to a guide stencil the prompt points at, the driver starts slimmer, and the orchestrator learns how to ask for a stop at a point.
`shed-template-driver.md` keeps its leading comment, markers and recipe-blind body, and now says: the one background job is `lyx shed step {{.run_id}} --until-stop`, launched as one bare line from the session's own cwd; the driver reads the envelope the job prints at its exit and never reads a trace, an artifact or a baseline status first; a job that ends without an envelope is re-run, and the re-run waits on a live loop or reports a dead one as an interrupted stop; `done` tears down with `{{.teardown_command}}`; `blocked`, `awaiting`, `paused` and `busy` keep their hand-backs and the park command; at an `error` or `interrupted` stop the driver spawns exactly one fork of itself with the envelope, and the fork reads `loop.trace_copy`, `loop.stderr_path`, the child traces and the repair guide at `{{.guide_path}}`, writes the stop report and any repair records, edits nothing else, and returns a short conclusion and one action (a named repair verb, a re-launch, or an escalation) that the driver then runs; a message asking for a stop at a point becomes `lyx shed pause {{.run_id}} --before <producer>` or `--after <producer>`, with producer names from the envelope's `progress` or `lyx shed status {{.run_id}}`, the driver answers the sender that the stop is recorded, and the stop itself reaches the parent by the existing notify rules; at a resume line over an `awaiting` run the driver launches the loop at once and never inspects the pull request or judges a decision, since the gate reads the recorded decision; the repair cap's key is the loop envelope's `current_producer` and `history_length`; the six error kinds and `loop.stop` are the branch fields.
The repair catalogue, the stranded-branch exception, the binary watch and the interrupted-invocation rule move verbatim into `shed-template-driver-guide.md`, a marker-free stencil registered in `entries` as `ShedTemplateDriverGuide`, which the driver stencil points at by its deployed path and never paraphrases; the guide is not an opening stencil and joins no role list.
Both stencils name no denied command and ask for no skill.
`driverPrompt` fills `driverGuideMarker` with `stencilstore.Path(stencilsDir, driverGuideStencilName)`, the deployed copy the fork reads; `driverSkills` becomes empty and `driverSpec` sets `ForkSubagents` true, so the claude engine permits fork subagents and still denies every other subagent type, with the reason stated in its comment.
`wordingClaims`' driver row keeps its existing claims and adds: must `--until-stop`, must `one fork`, must `--before`, must `--after`, must `never inspects the pull request`, must `{{.guide_path}}`, must not `read its trace_file right away`, must not `reading the artifact`, must not `Baseline`; a new row for the guide pins `Stranded-branch exception`, `Repair cap`, `Binary watch`, `Interrupted invocations` and that it names no marker.
`orch-template-role.md` gains one line under "Fixing a run": to stop a run at a point, message its driver with the producer name and whether to stop before or after it.
`driverprompt_test.go` asserts the guide marker is filled with the deployed path; `driverspec_test.go` asserts empty skills and `ForkSubagents` true.
`docs/overview.md`'s shed module paragraph on the driver stencil describes the loop command, the one-shot fork and the guide.

**ImpactSummary:** Prompt text and one marker in `driverPrompt`; `driverSpec` flips two fields the claude engine already honours; the stencil registry gains one marker-free entry.
