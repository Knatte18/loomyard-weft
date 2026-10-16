---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md` D5, settle whether each stated bound on the bypass open session is true in the code: `Runner.Open` in `internal/shuttleengine/open.go` adds the strand in a detached own window and returns at once, and `lyx shuttle send` in `internal/shuttlecli/send.go` types a turn into a live reed strand's pane by guid with no run. Question: does D5 state what an unwatched, `send`-driven bypass session can run, and what bounds it?"
  - "In the same D5, check that the hook, deny-rule and `--settings` claims match `buildOpenCmd` in `internal/shuttleengine/claudeengine/command.go` and the covering-pause refusal order in `Claude.OpenLaunch`. Question: does each bound name only what the code does?"
---

The focus targets the attack-surface question for the new bypass edge, the one place the first review found a bound stated without support in the code.
