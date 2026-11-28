# The dobby resumes lifted pauses, posts severe halts to Slack, keeps quiet on correct waits and chases unsettled stalls

`dobby-pause-lift-resume` · Task · feature, enhancement, dobby, tender, batten, slack, creel, shuttle, loom, config · draft

Decision record: [Sketch](decision-dobby-pause-lift-resume.md)

Depends on: [`driver-settles-stops`](design-driver-settles-stops.md), [`catchup-fetch-stops-start`](design-catchup-fetch-stops-start.md), [`kde-window-placement-build`](design-kde-window-placement-build.md), [`creel-machine-outbound-flake`](design-creel-machine-outbound-flake.md)

## The dobby resumes a run handed back for a pause once the pause lifts, and never a run the orch holds

Filed from the Sketch `self-healing-runs`.

Takes: D4's pause-lift and hold steps: `liftPending` and `spawn` in `internal/dobbyengine/passes.go`, `syncBoltHold` in `internal/dobbyengine/catchupwait.go`, `Pause` in `internal/tenderengine/pause.go` and `pauserecord.Entry`; the `--no-wait` flag from loom-verbs-await-park, which batten passes on the lift pass's child resume.
Delivers: The lift resumes a run whose last stop is the hand-back for the lifted pause, matched by scope, with a dobby pass on a run that is not drivable, keeping the lift in the dobby state record until the resume succeeds or the run moves on; a hold reason on a pause entry, set on a live entry of the scope when one exists, that stands through a `lym tender resume` with no slug and is lifted only by one naming the slug; a bolt-hold entry carrying a hold reason replaced by the dobby's own hold entry when it lifts the bolt-hold. The driver rows of D4 are built by driver-settles-stops.
Exports: Fixed for consumers: the hold reason field on `pauserecord.Entry` and the tender pause option that sets it, which the orch's hold of orch-unattended-rules uses.

The part changes these packages:

- internal/dobbyengine (about 480 changed lines)
- internal/tenderengine (about 220 changed lines)
- internal/pauserecord (about 50 changed lines)

## The dobby posts each severe halt to Slack, and the relay mentions the operator on a severe-halt post

Filed from the Sketch `self-healing-runs`.

Takes: D7's dobby steps and D8's relay steps: the four facts and the halt key stop-halt-count records on the history entry or the stop record; `ChildReading` in `internal/dobbyengine/runnotice.go`, read through `ReadChild` in `internal/battencli/dobbynotice.go`; the program post `internal/dobbyengine/waitreminder.go` makes, wired in `internal/beamcli/dobbydeps.go`; `internal/slackengine/outbound.go` and the operator ID in `slackengine.Setup`; `dobbyengine.AutonomyLevel` from autonomy-unattended-level.
Delivers: D7: `ChildReading` carries the four facts from the halt entry the stop record names, or from the stop record; the dobby that drives the run posts one `[operator] severe halt <slug> <key>:` line to `<shortname>:SLACK` under its program name for each `settle: operator` halt under `true` or `unattended`, whether or not an orch strand is recorded, and keeps the posted key in its state record, written after the post. D8: the relay mentions the configured operator on a post whose first line opens `[operator] severe halt <slug>`, from an orch or a program, and dedups a second one by slug and key within `slack.remind_after_min`, keyless only against keyless; every other post carries no mention. D8's reminders and D10's stall posts are built by dobby-stall-chase; the orch's relay rules by orch-unattended-rules.
Exports: The severe-post record in the dobby state record, which dobby-stall-chase's reminders and stall posts read.

The part changes these packages:

- internal/dobbyengine (about 420 changed lines)
- internal/battencli (about 70 changed lines)
- internal/beamcli (about 60 changed lines)
- internal/slackengine (about 260 changed lines)

## The idle and held notices skip correct waits and name what an agent waits on

Filed from the Sketch `self-healing-runs`.

Takes: D11 whole: the quiet notice in `internal/dobbyengine/runnotice.go`, `readAgentActivity` in `internal/battencli/wire.go`, `Store.AwaitsReply` in `internal/creelengine/verbs.go` and `creelcli.AwaitsReplyProbe`, the held notice in `internal/shuttleengine/hold.go` and `Runner.SetReplyProbe` in `internal/shuttleengine/run.go`, and `verifyBudgetNotice` in `internal/loomshed/verifybudget.go`.
Delivers: The quiet notice raises nothing for a `Raddle-Write` run whose proposal passes with a live raddle agent or for an agent with an open question in creel, and names an unread reply; `AwaitsReply` read as two probes for the quiet notice while the held notice keeps the combined one; an idle notice names the live command, its run time and its last output, or the Finalize step and what it waits on; the held notice is not sent while a question is open or a reply unread, in every runner; the verify gate's past-budget notice names the failing runs or says none failed.
Exports: Fixed for consumers: `creelcli.OpenQuestionProbe` and `creelcli.UnreadReplyProbe`, which dobby-stall-chase's question chase reads.

The part changes these packages:

- internal/dobbyengine (about 300 changed lines)
- internal/battencli (about 130 changed lines)
- internal/creelengine (about 130 changed lines)
- internal/creelcli (about 50 changed lines)
- internal/shuttleengine (about 160 changed lines)
- internal/loomshed (about 40 changed lines)

## The dobby chases stalls the orch leaves unsettled, reminds of severe posts, and warns when autonomy cannot reach the operator

Filed from the Sketch `self-healing-runs`.

Takes: D10 and D12 whole and D8's reminder steps: severe-halt-slack-post's severe-post record and post; `creelcli.OpenQuestionProbe` from idle-notice-correct-waits; the respawn cap the child read shows; the dobby `ConfigSpec` in `internal/dobbyengine/config.go`; the slack button state and the configured operator, read through the dobby's wiring in `internal/beamcli/dobbydeps.go`.
Delivers: D10: `beam.halt_chase_min`, default 10, classified `CategoryNone` and refused under 1 naming the key in full; one chase line per stall key to the dobby's orch for a `settle: orch` halt, an open creel question and a dead driver past its caps, naming `lym loom resume` when a parent decision came after the halt; under `unattended` one severe post per stall key one interval later, also sent to the orch while its slack button is off; chase and post kept in the state record and ended by D10's end events. D8: the wait reminders remind of each severe post the state record holds at `slack.remind_after_min`, without a mention, until its end event. D12: one warning per episode when a task resolves `true` or `unattended` with the slack button off or no operator configured, naming `lym slack`.
Exports: The `beam.halt_chase_min` key.

The part changes these packages:

- internal/dobbyengine (about 620 changed lines)
- internal/beamcli (about 90 changed lines)
- internal/battencli (about 40 changed lines)

