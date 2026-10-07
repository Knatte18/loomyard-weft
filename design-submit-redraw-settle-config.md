# The post-Enter redraw wait comes from shuttle.yaml

`submit-redraw-settle-config` · Note · enhancement, shuttle

`bugfix-sessions` (PR #417) moved the wait between pasted text and Enter into `shuttle.yaml` as `submit_settle_ms`, as the operator asked for that 300 ms constant.
The wait after Enter before the input box is read again, `submitRedrawSettle` in `internal/claudeengine/session.go`, is still a hardcoded 300 ms.
It should come from `shuttle.yaml` too, beside `submit_settle_ms`, read through `claudeengine.NewFromConfig`.

Found by the hub's PR-Gate reviewer on 2026-10-07; outside that task's goal, so not a reject.
