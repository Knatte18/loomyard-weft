# beam.autonomy takes false, true or unattended, and every reader reads the level

`autonomy-unattended-level` · Task · feature, dobby, config, cwd, board, orch, slack, loom · running · Darn-Write

Decision record: [Sketch](decision-autonomy-unattended-level.md)

Depends on: [`beam-orch-reads-down`](design-beam-orch-reads-down.md)

Filed from the Sketch `self-healing-runs`.

Takes: D1 whole: `LookupAutonomy` in `internal/dobbyengine/tasksetting.go`, `checkTaskSetting` in `internal/dobbyengine/config.go` and its callers, the template comment in `internal/dobbyengine/template.yaml`, `hubgeom.Autonomy` in `internal/hubgeom/autonomy.go` and its callers in slackcli, loomcli, orchcli and beamcli, the beam modes in `internal/boardengine/beammodes.go`, and the autonomy line in `internal/orchengine/prompt.go`, which also serves D9's rule that the line names the level.
Delivers: The three levels on every path that checks the key, a refusal naming the three values and `lym config beam`, a refused value read as `false` and posted once per episode to the orch, the level on the `autonomy:` stamp, `off`, `on` or `unattended` in the beam modes, and the level named in the orch prompt and the template comment. Every reader that asks only whether autonomy is on reads `true` and `unattended` as on.
Exports: Fixed for consumers: `dobbyengine.AutonomyLevel`, a string type with `AutonomyOff` ("false"), `AutonomyOn` ("true") and `AutonomyUnattended` ("unattended") and an `On() bool` method; `dobbyengine.LookupAutonomy` and `hubgeom.Autonomy` return it.

The part changes these packages:

- internal/dobbyengine (about 260 changed lines)
- internal/hubgeom (about 70 changed lines)
- internal/boardengine (about 40 changed lines)
- internal/orchengine (about 30 changed lines)
- internal/orchcli (about 40 changed lines)
- internal/slackcli (about 15 changed lines)
- internal/loomcli (about 30 changed lines)
- internal/beamcli (about 45 changed lines)
