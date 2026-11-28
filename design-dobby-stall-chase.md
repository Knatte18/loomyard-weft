# The dobby chases stalls the orch leaves unsettled, reminds of severe posts, and warns when autonomy cannot reach the operator

`dobby-stall-chase` · Task · feature, dobby, batten, config · draft

Decision record: [Sketch](decision-dobby-stall-chase.md)

Depends on: [`idle-notice-correct-waits`](design-idle-notice-correct-waits.md)

Filed from the Sketch `self-healing-runs`.

Takes: D10 and D12 whole and D8's reminder steps: severe-halt-slack-post's severe-post record and post; `creelcli.OpenQuestionProbe` from idle-notice-correct-waits; the respawn cap the child read shows; the dobby `ConfigSpec` in `internal/dobbyengine/config.go`; the slack button state and the configured operator, read through the dobby's wiring in `internal/beamcli/dobbydeps.go`.
Delivers: D10: `beam.halt_chase_min`, default 10, classified `CategoryNone` and refused under 1 naming the key in full; one chase line per stall key to the dobby's orch for a `settle: orch` halt, an open creel question and a dead driver past its caps, naming `lym loom resume` when a parent decision came after the halt; under `unattended` one severe post per stall key one interval later, also sent to the orch while its slack button is off; chase and post kept in the state record and ended by D10's end events. D8: the wait reminders remind of each severe post the state record holds at `slack.remind_after_min`, without a mention, until its end event. D12: one warning per episode when a task resolves `true` or `unattended` with the slack button off or no operator configured, naming `lym slack`.
Exports: The `beam.halt_chase_min` key.

The part changes these packages:

- internal/dobbyengine (about 620 changed lines)
- internal/beamcli (about 90 changed lines)
- internal/battencli (about 40 changed lines)
