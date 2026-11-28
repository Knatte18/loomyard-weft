# lym loom resume and lym loom start wait for the driver to park, with a no-wait option batten passes

`loom-verbs-await-park` · Task · enhancement, loom, batten · draft

Decision record: [Sketch](decision-loom-verbs-await-park.md)

Depends on: [`autonomy-unattended-level`](design-autonomy-unattended-level.md)

Filed from the Sketch `self-healing-runs`.

Takes: D5 whole: the `shedrun.StartNotParkedKind` refusals in `internal/loomcli/resume.go` and `internal/loomcli/start.go`, the driver's wait record `sheddriver.ReadDriveWait` that push-retry-not-halt added, the keep-warm cap from `shuttleengines.KeepWarm`, `childSpawnError` in `internal/battencli/wire.go` and the `battenshed.ErrChildNotParked` retry in `internal/battenshed/innerrun.go`.
Delivers: Both verbs wait for the park, or for a run lock on a non-running run to be released, under the keep-warm cap, waking at most once a second with an injected clock; answer at once for a driver that keeps a wait record; refuse past the cap naming the retry; and refuse at once with the no-wait option, which batten passes on its child start and on every resume it runs, the resume of dobby-pause-lift-resume's lift pass among them.
Exports: Fixed for consumers: the `--no-wait` flag on `lym loom resume` and `lym loom start`.

The part changes these packages:

- internal/loomcli (about 420 changed lines)
- internal/battencli (about 60 changed lines)
- internal/battenshed (about 50 changed lines)
