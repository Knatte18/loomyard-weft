# The skill-load hold landed with two checks by hand left open

`skill-load-hold-unchecked` · Task · bug, shuttle · running · Drift-Await

## Seen

The landing of `skill-load-timeout-sends-prompt` (`23518a94`) holds a session whose skill-load turn the API failed, instead of typing its prompt.
Nothing was run by hand and no regression check ran; its commit lists two checks left open:

- `hold-behaviors-lack-pinning-rows`: whether the rows of `TestStartup_LoadTurnAPIError` in `internal/shuttleengine/startup_test.go` pin every hold behavior: back-off doubling and ceiling, busy session, turn start during the back-off, dying pane, failed send, notice once and none when interactive, notifier failure, model rejected, open turn at the deadline, interrupted turn.
- `open-load-turn-skips-and-types-prompt`: whether `loadSkillTurn` in `internal/shuttleengine/loadturn.go` neither skips its skills nor lets the prompt be typed for an open load turn past the timeout, and whether that test pins it.

## Idea

Read the two against the code and add a row for any behavior left unpinned; no real launch is needed for these two.
