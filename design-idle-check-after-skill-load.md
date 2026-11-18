# shuttle start fails with 'session is not idle' when the pane shows an idle input prompt after skill loading

`idle-check-after-skill-load` · Note · bug, shuttle

Imported from [issue #137](https://github.com/Knatte18/loomyard/issues/137).

## What happened

A `burler-review` seat (`code-burler-review`) failed to start twice in a row, and the loom run halted as failed with anomaly `producer-hard-failure` ("two consecutive died/timeout outcomes", attempts 1 and 1b).

Both attempts died in `shuttle` with: `deliver the prompt after loading skills: the session is not idle: strand "...": no input box is on screen; a menu or picker may be open; retry once the session is idle`, followed by `the provider never became ready`.

The pane capture embedded in the error shows the session had finished loading every skill and printed its `ok <skill names>` reply and a `done` timing line.
The last lines were an empty `❯` prompt and the `bypass permissions on` footer, which looks like an idle session with an input box.
No menu or picker is visible in the capture.

## Why it is abnormal

The session was ready for input, yet the idle check reported no input box.
Both attempts failed the same way within seconds of the skill replies (3 s and 4 s turns), so this looks like a race or a mismatch in the idle detection rather than a stuck session.
Possible causes: the check ran in the moment between the reply finishing and the prompt box being redrawn, or the check does not recognize this layout (the `✻ ... · done` line directly above the prompt).
"Retry once the session is idle" is not acted on by the seat engine: the strand is removed and the attempt counts as died, so one transient detection miss costs a full attempt and two of them halt the run.

## What to do

- Check the idle detection in the `shuttle` prompt delivery step against the saved `startup-capture.txt` of the two run dirs (`.lym/shuttle/f31a8e1cc4fffddafce3cadb9a61d6f7` and `.lym/shuttle/6c6752222111cdd03ba84620de79c3f6`).
- Make the delivery step poll for the input box for a bounded time before declaring the session not idle, instead of failing on the first miss.
- Consider that a `not idle` refusal that names "retry once idle" is retried by `shuttle` itself rather than killing the strand.

Provenance: task slug `inmemory-store-windows-root`; notes: `loom-halt.md`.
