# Shuttle idle check reports no input box on an idle seat, killing Plan-Burler-Review

`seat-idle-no-input-box` · Note · bug, shuttle, claude

Imported from [issue #219](https://github.com/Knatte18/loomyard/issues/219).

## What happened

The `Plan-Burler-Review` row (`burler-review`) halted `failed` (anomaly `producer-hard-failure`) after round 3: two consecutive `died` outcomes, both "review never started".
Each attempt's seat start ended with:

```
shuttle: deliver the prompt after loading skills: shuttle: the session is not idle: strand "<id>": no input box is on screen; a menu or picker may be open; retry once the session is idle.
```

The pane lines quoted in the same error show the opposite: the skill-load reply `ok scribe:prose, scribe:code-quality, scribe:testing` had finished, a `done` status line was printed, and the `❯` input prompt plus the `bypass permissions on` footer were on screen.
The session was idle with an input box; the idle check reported none.
It happened on attempts 3 and 3b, in two separate halts, with the same pane text each time (`Sautéed for 3s`, `Cogitated for 3s`, `Cooked for 2s` status lines).

## Why abnormal

The seat was healthy.
A false "no input box" reading kills a seat that never got its prompt, burns the review rounds, and halts the whole run as `failed`.
The error text itself says "retry once the session is idle", yet the shuttle did not retry after the capture showed an idle pane.

## What to do

Check the input-box detection in the shuttle's deliver-after-skills step against a pane whose last lines are a completed skill-load reply, a `✻ <verb> for Ns · done` line, the `❯` prompt and the permissions footer.
Retry the idle check for a short window before declaring `died`, so a repaint race right after the skill-load reply does not kill the seat.

Provenance: task `models-torque-drag-adapter`; notes `loom-halt-2.md`, `loom-halt-3.md`.
