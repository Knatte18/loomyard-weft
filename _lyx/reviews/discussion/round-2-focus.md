---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md` D4, settle whether the pane-tail trigger matches the code: `loadSkillsThenPrompt` in `internal/shuttleengine/run.go` calls `abandonStartup(OutcomeDied, nil)` after ready, while `awaitStartup` in `wait.go` has the never-ready exits; does the stated trigger and the acceptance criterion pick the same set of teardowns, and does the cause-present sentence describe only causes that carry a pane tail?"
  - "In D5 and the Constraints line for `PATTERN-daemon-wakeups`, settle whether the stated reason `awaitStartup` is outside the rule agrees with `pattern/PATTERN-daemon-wakeups.md` (its definition of an unbounded wait and its treatment of a bounded wait), given `startup_timeout_s` in `internal/shuttleengine/template.yaml` and a one-second probe that captures a pane."
  - "In D6, settle whether the dismissal gap's behavior is fully stated at its boundaries: the floor applied to `liveness_every_n_polls`, a probe exactly one gap after the last dismissal, and what counts as a played dismissal."
---

Focus targets the design-class questions still open at the end of round 1.
