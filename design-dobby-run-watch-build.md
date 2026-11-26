# Dobby run watch and gate-hold notice: mail the parent on every stop and on a slot held too long

`dobby-run-watch-build` · Task · feature, dobby, gate, batten, orch, pattern · starting · Open-IDE

Decision record: [Sketch](decision-dobby-run-watch-build.md)

Depends on: [`dotnet-gate-fixes`](design-dotnet-gate-fixes.md)

Filed from the Sketch `dobby-run-watch`.

Takes: the Sketch's decision record; the dobby tick's run steps and listedRuns (internal/dobbyengine/loop.go), the landing notice and its per-run memory (landnotice.go, state.go), the done-run clear (donerun.go), RunNoticeDeps.ReadChild, the per-task lookup over hubgeom.TaskSettingChain (tasksetting.go), orchcli.NotifyRunFrom and WithRunAutonomy, gateslot.Pool.Holders and LoadConfig, gatecli's runMarker, proc.TreeSample and PIDsWithEnv, pauserecord.AwakeSince
Delivers: a run watch step in the dobby tick posting [run watch] lines per the run_watch level, with its per-run memory, owed lines and gate-hold keys in a new watch map of the dobby state record; beam.run_watch as a per-task key with its check; a probe-tick gate-hold check with gate_hold_factor and gate_hold_min hub keys and its [dobby notice] line; the tier run's environment marker in the slot holder record; orch-role Notices, dobby doc and template, and PATTERN-config-layers naming the new lines and keys
Exports: dobbyengine.LookupRunWatch, beamsetting's RunWatchKey and its three values, the dobby state record's watch map, gateslot.Holder's marker field

The part changes these packages:

- internal/dobbyengine (about 750 changed lines)
- internal/beamsetting (about 20 changed lines)
- internal/beamcli (about 120 changed lines)
- internal/battencli (about 40 changed lines)
- internal/gateslot (about 20 changed lines)
- internal/gatecli (about 30 changed lines)
- contracts (about 20 changed lines)
- pattern (about 5 changed lines)
