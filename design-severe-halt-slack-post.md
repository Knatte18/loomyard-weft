# The dobby posts each severe halt to Slack, and the relay mentions the operator on a severe-halt post

`severe-halt-slack-post` · Task · feature, dobby, batten, slack · draft

Decision record: [Sketch](decision-severe-halt-slack-post.md)

Depends on: [`driver-settles-stops`](design-driver-settles-stops.md), [`dobby-pause-lift-resume`](design-dobby-pause-lift-resume.md)

Filed from the Sketch `self-healing-runs`.

Takes: D7's dobby steps and D8's relay steps: the four facts and the halt key stop-halt-count records on the history entry or the stop record; `ChildReading` in `internal/dobbyengine/runnotice.go`, read through `ReadChild` in `internal/battencli/dobbynotice.go`; the program post `internal/dobbyengine/waitreminder.go` makes, wired in `internal/beamcli/dobbydeps.go`; `internal/slackengine/outbound.go` and the operator ID in `slackengine.Setup`; `dobbyengine.AutonomyLevel` from autonomy-unattended-level.
Delivers: D7: `ChildReading` carries the four facts from the halt entry the stop record names, or from the stop record; the dobby that drives the run posts one `[operator] severe halt <slug> <key>:` line to `<shortname>:SLACK` under its program name for each `settle: operator` halt under `true` or `unattended`, whether or not an orch strand is recorded, and keeps the posted key in its state record, written after the post. D8: the relay mentions the configured operator on a post whose first line opens `[operator] severe halt <slug>`, from an orch or a program, and dedups a second one by slug and key within `slack.remind_after_min`, keyless only against keyless; every other post carries no mention. D8's reminders and D10's stall posts are built by dobby-stall-chase; the orch's relay rules by orch-unattended-rules.
Exports: The severe-post record in the dobby state record, which dobby-stall-chase's reminders and stall posts read.

The part changes these packages:

- internal/dobbyengine (about 420 changed lines)
- internal/battencli (about 70 changed lines)
- internal/beamcli (about 60 changed lines)
- internal/slackengine (about 260 changed lines)
