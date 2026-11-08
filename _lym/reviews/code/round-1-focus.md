---
round: 1
focus:
  - "Test-binary loop: every place the diff pins a command from `executablePath` (reed status job and toggle click in `internal/reedengine`, hook commands in `internal/shuttleengine/claudeengine/settings.go`) — can any of them run a test binary under `go test`, and does the `TestMain` guard plus the test that proves a test-session status job never launches the test binary cover it?"
  - "Presence is per machine: `internal/slackengine/presence.go`, `switch.go`, `relay.go` — does away on one hub hold for every hub and orch on the machine, and are reads and writes of the state consistent across hubs?"
  - "Auto-switch to away: the relay sets presence before it forwards an operator message, only when connected and presence is `terminal`, and the `[slack presence]` notice goes out as for any other change (`inbound.go`, `relay.go`)."
  - "Auto-switch back to terminal in `internal/slackcli/hook.go` and `internal/shuttleengine`: how the operator's own typed lines are told from slash commands, creel injects (matched against the creel journal), task notifications and hook mail; is each kind tested, and are failure paths (missing journal, race with the inject) handled?"
  - "Stop hook: forwards only the turn's final reply to that orch's channel while away; sends nothing for a turn with no text; covers automated-notice turns; check the `claudeengine` settings wiring and the hook command's failure paths."
  - "Reed toggles (`slacktoggle.go`, `panebin.go`, `windowsize.go`): labels `slack` (connection) and `away` (presence), lit state, redraw on a change by any route, and a click that starts or stops the relay or switches presence."
  - "Sender names posted to Slack carry the machine in the `<machine>::<sender>` form (`outbound.go`, `geometry.go`)."
  - "Scope against the decision record: `role-slack.md` trimmed to one plain line on sending through creel; check whether the diff makes changes the record does not license."
  - "Claims in `_lym/landing/summary.md` checked against the diff, and the commit messages of the range."
  - "Test economy: new tests that duplicate existing coverage, per-helper tests already covered by public-surface tests, and behaviours added with no test."
---

Seed focus drawn from the decision record's Goal, its two amendments and the diffstat of the range.
