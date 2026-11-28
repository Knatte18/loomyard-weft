# beam.autonomy takes false, true or unattended

`beam.autonomy` was a boolean.
It now takes three levels, to make room for a stricter overnight mode:

- `false`: the orch relays to the operator every halt it does not settle, as before.
- `true`: autonomy, as before.
- `unattended`: meant as the level where the orch settles every halt and never asks the operator,
  and only a severe halt reaches them.
  The orch prompt line and the template comment say so; the orch role texts do not yet define the handling.

What changes with the third level:

- The lookup, the beam config load, `lym config beam --set` and a task's `run_settings` accept all three.
  Any other value is refused, naming the three and `lym config beam`.
- A halt notice's `autonomy:` stamp carries the level.
- The board's beam modes show `off`, `on` or `unattended`.
- The orch prompt's autonomy line names `unattended` when that is the level.
- The template comment on `beam.autonomy` describes all three.
- Readers that ask only whether autonomy is on (loom's run and pawl wiring, the Slack wiring) treat `true` and `unattended` as on.
- A refused value still reads as `false`.
  The dobby now also posts the refusal to its orch, once per episode, when it stamps one of its own mails about that run.

## Design decisions

- `false` and `true` keep their meaning, so no config reinterpretation ships and a boolean in a board `run_settings` stays valid.
- `dobbyengine.LookupAutonomy` and `hubgeom.Autonomy` return a `dobbyengine.AutonomyLevel` instead of a bool.
  `On()` answers the on-or-off question.
- The level strings are declared once, in `beamsetting`.
  `orchengine` has its own `AutonomyLevel`, which `orchcli` maps to.
- `hubgeom.AutonomyWithRefusal` hands the refusal to the dobby.
  A read that fails for another reason neither posts nor ends an episode.
- The refusal is posted without holding the lock, so another mail of the same run stamped meanwhile may arrive first.
  A failed post is retried at the run's next refused read.

## What the tests pin

- The lookup, the beam config load and the local values check accept `false`, `true` and `unattended`.
  They refuse any other value, naming the three and `lym config beam`.
- `hubgeom` returns the level and reads a refused value as `false`.
- A dobby mail's stamp carries the level, and a refusal is posted once per episode.
- The board's beam modes show `unattended`.

## To check by hand

- The orch role texts under `contracts/orchrole/` are unchanged and do not mention `unattended`.
  `role-autonomy.md` and `orch-role.md` tie autonomy handling to a notice carrying `autonomy: true`,
  while the prompt line for `unattended` points the orch to `role-autonomy.md`.
  Check that this is acceptable until the role texts describe the level, or hold off setting `unattended`.
- Check that `TestRunAutonomyStamps_PostsARefusalOncePerEpisode`
  (`internal/beamcli/dobbyautonomystamps_test.go`) opens with `tiergate.Require(t, 0)`,
  as `TestTierTags_OneTierPerTestFile` in `cmd/lym` demands.
- Halt notices that loom and batten post through `NotifyParentHalt` and `NotifyRunFromHalt`
  (`internal/orchcli/notify.go`) stamp a refused value as `false` and drop the refusal.
  The orch learns of it only when the dobby stamps one of its own mails about the run,
  so under `beam.run_watch: off` it may never learn why.
  Decide whether that gap is acceptable.

## Merge danger

**Door:** one-way
**Blast radius:** config

A build older than this change refuses the beam config while a layer holds `unattended`, and so does a revert of it.
Nothing in the change writes that value; the risk starts when someone sets it.
Set it only once every machine of the hub runs a build that knows it, and remove it before reverting.
