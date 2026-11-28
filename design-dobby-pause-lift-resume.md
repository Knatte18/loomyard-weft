# The dobby resumes a run handed back for a pause once the pause lifts, and never a run the orch holds

`dobby-pause-lift-resume` · Task · feature, dobby, tender · draft

Decision record: [Sketch](decision-dobby-pause-lift-resume.md)

Depends on: [`catchup-fetch-stops-start`](design-catchup-fetch-stops-start.md), [`kde-window-placement-build`](design-kde-window-placement-build.md)

Filed from the Sketch `self-healing-runs`.

Takes: D4's pause-lift and hold steps: `liftPending` and `spawn` in `internal/dobbyengine/passes.go`, `syncBoltHold` in `internal/dobbyengine/catchupwait.go`, `Pause` in `internal/tenderengine/pause.go` and `pauserecord.Entry`; the `--no-wait` flag from loom-verbs-await-park, which batten passes on the lift pass's child resume.
Delivers: The lift resumes a run whose last stop is the hand-back for the lifted pause, matched by scope, with a dobby pass on a run that is not drivable, keeping the lift in the dobby state record until the resume succeeds or the run moves on; a hold reason on a pause entry, set on a live entry of the scope when one exists, that stands through a `lym tender resume` with no slug and is lifted only by one naming the slug; a bolt-hold entry carrying a hold reason replaced by the dobby's own hold entry when it lifts the bolt-hold. The driver rows of D4 are built by driver-settles-stops.
Exports: Fixed for consumers: the hold reason field on `pauserecord.Entry` and the tender pause option that sets it, which the orch's hold of orch-unattended-rules uses.

The part changes these packages:

- internal/dobbyengine (about 480 changed lines)
- internal/tenderengine (about 220 changed lines)
- internal/pauserecord (about 50 changed lines)
