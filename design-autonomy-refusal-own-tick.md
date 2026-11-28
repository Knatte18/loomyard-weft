# Dobby posts a refused beam.autonomy on its own tick

`autonomy-refusal-own-tick` · Note · bug, dobby

## Seen

In the run of `autonomy-unattended-level`, Code-Review round 1 finding F7:
the halt notices loom and batten post through `NotifyParentHalt` and `NotifyRunFromHalt` stamp a run's autonomy through `hubgeom.Autonomy`, which drops the refusal.
The orch hears of a refused `beam.autonomy` value only when the dobby builds one of its own stamped mails about the run.
Under `beam.run_watch` off, a run whose halts come only from loom or batten is stamped `false`, and its refusal is never posted.

## Idea

The dobby checks each listed run's `beam.autonomy` value on its own tick, whether or not it posts a mail,
and posts a refusal through the same `runAutonomyStamps` episode, so a mistyped level never passes unseen.
